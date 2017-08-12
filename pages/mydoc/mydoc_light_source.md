---
title: Light Source
summary : "A simple point light source"
sidebar: mydoc_sidebar
permalink: mydoc_light_source.html
folder: mydoc
toc : true
---

## Overview
I think it's time to spruce up our simulation a little. 
Up until this point we've been making due with omipresent, omidirectional, uniform strength light. 
In this article we will be removing the first inaccuracy by creating a uniform strength light source that radiates light out in all directions from it's center point.
The meat of this article is taken directly from [learnopengl](https://learnopengl.com/#!Lighting/Basic-Lighting).
Instead of repeating it all here I'm going to focus on the specifics of implementing it within the framework we've built so far.
I suggest you take the time to read the original article.

## Light source
We will be adding a struct that represents the light source described in the overview.  
```c++
#ifndef LIGHT_H
#define LIGHT_H
#include <vector3G.h>

struct Light
{
    public :
        Vector3Gf position;
        Vector3Gf ambient;
        Vector3Gf diffuse;
        Vector3Gf specular;

    Light();

    Light(Vector3Gf position, Vector3Gf ambient, Vector3Gf diffuse, Vector3Gf specular);
};
#endif
```
The cpp file just assigns the constructor parameters to the member functions.

## Scene
Scene will be modified to store a reference to the light source for the simulation.
```c++
class Scene
{
    private:
        std::vector<std::shared_ptr<PhysicsEntity>> m_physics_entity_ptrs;
        std::vector<std::shared_ptr<Model>> m_model_ptrs;
        std::shared_ptr<TimeIntegrator> m_time_integrator;

        NetForceAccumulator m_net_force_accumulator;

        Light m_light;

    public: 

    // Same as before

    /**
        Sets the light for the scene
        @param light the light source
    **/
    void SetLight(Light light);

```
The cpp file just implements the setter for the light source

## Normals
In order to compute diffuse lighting we'll be needing normal vectors for each vertex in our model.
In principle they can be computed at runtime as described [here](https://stackoverflow.com/questions/6656358/calculating-normals-in-a-triangle-mesh/6661242#6661242)
However they are usually computed once and save in the model file.
Since we haven't been saving our model file but generating the vertices at runtime (which is only because I've been too lazy to deal with file IO yet) we'll be leaning more towards computing them at runtime. 
However since we are dealing with a specific geometric shape, a sphere, we can use this a-priori knowledge to simplify the calculation.
With a moments consideration it should become clear that the normal to point on a sphere is the vector connecting that point to the center of the sphere. 
Since our spheres are centered at the origin in local coordinates that means the point representation of the vertex is actually the (unnormalized) normal vector. 

### Mesh
First we update *Mesh* to store a list of vertex normals

```c++
class Mesh
{
    private:
        List3df m_vertices; // Matrix of vertices in mesh
        List3di m_faces; // Matrix of indices into vertex list in counter-clockwise order.
        List3df m_normals; // Matrix of vertex normals

        // Same as before

    public:
        /**
            Builds a mesh with no vertices or edges
        **/
        Mesh();

        /**
            Builds a mesh with the provided vertices and edges
            @param vertices nx3 matrix of vertices for the mesh. 
            @param faces mx3 matrix of face indices for the mesh. Listed in counter-clockwise order
            @param normals mx3 matrix of vertex normals
        **/
        Mesh(List3df vertices, List3di faces, List3df normals);

        /**
            @return A copy of the face normals in the mesh
        **/
        List3df GetNormals();

        // Same as before
};
```
Next we'll update *GenerateVAO* to move the vertex normals to the GPU.
We'll be adopting the convention that each vertex is immediately followed by its normal in the memory block.
Meaning each "vertex" is actually a block of 6 floats, 3 for the position and 3 for the normal.
Accomplishing this will require us to update our **Vertex Array Pointer**.
In fact we'll be splitting into 2 pointers, one for vertices and one for normals.
This involves replacing 
```c++
Eigen::Map<List3df>( vertices, m_vertices.rows(), m_vertices.cols() ) = m_vertices;
```
with
```c++
GLuint n_V = m_vertices.rows();
GLfloat vertices[6*n_V];
for(int i=0; i < n_V; i++)
{
    vertices[6*i]   = m_vertices(i,0);
    vertices[6*i+1] = m_vertices(i,1);
    vertices[6*i+2] = m_vertices(i,2);
    vertices[6*i+3] = m_normals(i,0);
    vertices[6*i+4] = m_normals(i,1);
    vertices[6*i+5] = m_normals(i,2);
}
```
and replacing

```c++
glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3*sizeof(GLfloat), (GLvoid*)0);
glEnableVertexAttribArray(0);
```

with
```c++
// vertex positions
glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 6*sizeof(GLfloat), (GLvoid*)0);
glEnableVertexAttribArray(0);

// vertex normals
glVertexAttribPointer(1, 3, GL_FLOAT, GL_FALSE, 6*sizeof(GLfloat), (GLvoid*)(3*sizeof(GLfloat)));
glEnableVertexAttribArray(1);

```

### Sphere
As we mentioned earlier, the normals for a sphere centered on the origin are just the vertices themselves.
Therefore we just need to update $$UVSphereMesh$$ to pass the vertex matrix as the normal matrix
```c++
mesh = Mesh(vertices,faces,vertices);
```
 
## Normal Matrix

The normal matix handles transforming the normal vectors from local coorindates to world coordinates. 
It is the transpose inverse of the upper $$3 \times 3$$ block of the model matrix.
Since inverting a matrix is an expensive operation we'll perform it once and store it in the *Model* class.

### Model
```c++
class Model {
   public:

        // Same as before
        
        /**
            @return The normal matrix
        **/
        Eigen::Matrix<GLfloat,3,3> GetNormalMatrix();

     protected:

        // Same as before

        /**
            The normal matrix. Used in rendering light interaction with surface
        **/
        Eigen::Matrix<GLfloat,3,3> m_normal_matrix;
};
#endif
```

```c++
Model::Model()
{
        m_model_matrix.setIdentity();
        m_normal_matrix.setIdentity();
}

void Model::SetModelMatrix(Eigen::Matrix<GLfloat,4,4> m)
{
    m_model_matrix = m;
    m_normal_matrix = m.block(0,0,3,3).inverse().transpose();
};

Eigen::Matrix<GLfloat,3,3> Model::GetNormalMatrix()
{
    return m_normal_matrix;
}
```

### Vertex Shader
We'll be updating this to pass forward the normal vector and fragment position in world coordinates.
Notice that we now have 2 inputs corresponding to our 2 Vertex Array Pointers.

```c++
#version 330 core
layout (location = 0) in vec3 position;
layout (location = 1) in vec3 normal;

uniform mat4 model;
uniform mat4 view;
uniform mat4 projection;
uniform mat3 normalMat;

out vec3 Normal;
out vec3 FragPos; // fragment position in world coordinates

void main()
{
    gl_Position = projection*view*model*vec4(position, 1.0f);
    Normal = normalMat*normal;
    FragPos = vec3(model*vec4(position,1.0f));
}
```
### Fragment Shader
In the fragment shader we'll be computing the effect of the light source on that fragment. 
The details of this calculation are specified in the article linked at the top.

```c++
#version 330 core
struct Light{
    vec3 position;

    vec3 ambient;
    vec3 diffuse;
    vec3 specular;
};

in vec3 Normal;
in vec3 FragPos;

out vec4 color;

uniform vec3 viewPos;
uniform Light light;

void main()
{

    //Ambient
    vec3 ambient = light.ambient;

    // Diffuse
    vec3 n = normalize(Normal);
    vec3 lightDir = normalize(light.position - FragPos);
    float diff = max(dot(n,lightDir),0.0f);
    vec3 diffuse = diff * light.diffuse;

    // Specular
    vec3 viewDir = normalize(viewPos - FragPos);
    vec3 reflectDir = reflect(-lightDir,n);
    float spec = pow(max(dot(viewDir,reflectDir),0.0f),material.shininess);
    vec3 specular = spec * light.specular;

    vec3 result = ambient + diffuse + specular;
    color = vec4(result,1.0f);
}
```

## Results
And we're done.
Honestly that turned out to be a lot more than I expected. 
But at least we got to refamilarize ourselves with some of the lower level entities of our engine.

{% include links.html %}
