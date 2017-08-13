---
title: Material
summary : "Adding material properties of to a model"
sidebar: mydoc_sidebar
permalink: mydoc_material.html
folder: mydoc
toc : true
---

## Overview
Up until this point all the entities in our scene have been constrained to look the same. 
This is naturally very limiting as it means all entities must have the same color, reflectivity, etc.
In this article we will be adding a *Material* property to our models that will allow them to differentiate themselves from eachother.
Again we will be basing this on a learnopengl [article](https://learnopengl.com/#!Lighting/Materials)

## Material
The material struct will look almost exactly like a light source, but it lacks a poisition and has a "shininess" attribute.
This second attribute controls the intensity of the specular reflection.

```c++
#ifndef MATERIAL_H
#define MATERIAL_H
#include <vector3G.h>

struct Material
{
    public :
        Vector3Gf ambient;
        Vector3Gf diffuse;
        Vector3Gf specular;
        GLfloat shininess;

        Material();

        Material(Vector3Gf ambient,Vector3Gf diffuse,Vector3Gf specular,GLfloat shininess);
};

#endif
```

## Model
Each model will now contain a reference to a material instance
```c++
class Model {
   public:

        // Same as before

        /**
            #return material for this mesh
        **/
        Material GetMaterial();

    protected:
        
        // Same as before

        /**
            Material properties for this model
        **/
        Material m_material;
```

## Sphere
We will need to be able to set the material for our sphere so we'll change our constructor signiture.

```c++
Sphere::Sphere(GLfloat radius, Vector3Gf position, Vector3Gf velocity, GLfloat mass, Material material) : Model(), PhysicsEntity()
{
    m_radius = radius;
    m_center = position;

    m_position = position;
    m_velocity = velocity;
    m_mass = mass;

    m_model_matrix.col(3) << position(0), position(1), position(2), 1.0f;
    UVSphereMesh(m_radius, 20, 20, m_mesh); // model matrix handles translation of mesh so we use (0,0,0) as mesh center

    m_material = material;
};
```

## Fragment Shader
The interplay between the light source and the material properties will determine how each fragment is rendered.
In most cases this will just mean taking the coefficent-wise product between the different light properties and their corresponding material properties.
The one exception being shininiess, which is used as an exponent.

```c++
#version 330 core
struct Material{
    vec3 ambient;
    vec3 diffuse;
    vec3 specular;
    float shininess;
};

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
uniform Material material;
uniform Light light;

void main()
{

   //Ambient
    vec3 ambient = material.ambient * light.ambient;

    // Diffuse
    vec3 n = normalize(Normal);
    vec3 lightDir = normalize(light.position - FragPos);
    float diff = max(dot(n,lightDir),0.0f);
    vec3 diffuse = material.diffuse * diff * light.diffuse;

    // Specular
    vec3 viewDir = normalize(viewPos - FragPos);
    vec3 reflectDir = reflect(-lightDir,n);
    float spec = pow(max(dot(viewDir,reflectDir),0.0f),material.shininess);
    vec3 specular = material.specular * spec * light.specular;

    vec3 result = ambient + diffuse + specular;
    color = vec4(result,1.0f);
}
```

## Scene
Lastly we'll need to set the *Material* uniform in our render loop.
I'll include the whole *Render* function since we've made some big changes between the last 2 articles.
When each model is loaded we grab it's *Material* instance and set the *Material* in the shader to match.
```c++
void Scene::Render(Shader shader, Vector3Gf view_pos)
{
    glUniform3f(glGetUniformLocation(shader.Program,"light.ambient"),m_light.ambient(0),m_light.ambient(1),m_light.ambient(2));
    glUniform3f(glGetUniformLocation(shader.Program,"light.diffuse"),m_light.diffuse(0),m_light.diffuse(1),m_light.diffuse(2));
    glUniform3f(glGetUniformLocation(shader.Program,"light.position"),m_light.position(0),m_light.position(1),m_light.position(2));

    glUniform3f(glGetUniformLocation(shader.Program,"viewPos"),view_pos(0), view_pos(1), view_pos(2));

    GLint modelLoc = glGetUniformLocation(shader.Program, "model");
    GLint normalLoc = glGetUniformLocation(shader.Program, "normalMat");

    const GLuint model_count = GetModelCount();
    std::shared_ptr<Model> model_ptr;
    for(int i = 0; i < model_count; ++i)
    {
        GLuint VAO;

        GetModel(i,model_ptr,VAO);

        Eigen::Matrix<float,4,4> model_matrix = model_ptr->GetModelMatrix();
        glUniformMatrix4fv(modelLoc, 1, GL_FALSE, model_matrix.data());

        Material material = model_ptr->GetMaterial();

        glUniform3f(glGetUniformLocation(shader.Program,"material.ambient"),material.ambient(0),material.ambient(1),material.ambient(2));
        glUniform3f(glGetUniformLocation(shader.Program,"material.diffuse"),material.diffuse(0),material.diffuse(1),material.diffuse(2));
        glUniform3f(glGetUniformLocation(shader.Program,"material.specular"),material.specular(0),material.specular(1),material.specular(2));
        glUniform1f(glGetUniformLocation(shader.Program,"material.shininess"),material.shininess);

        Eigen::Matrix<GLfloat,3,3> N = model_ptr->GetNormalMatrix();

        glUniformMatrix3fv(normalLoc, 1, GL_FALSE, N.data());

        glBindVertexArray(VAO);

        glDrawElements(GL_TRIANGLES, model_ptr->GetMesh().GetNumEdges(), GL_UNSIGNED_INT,0);

        glBindVertexArray(0);

    }
}
```

## Results
Now that we can differentiate our spheres lets make one blue and one red. 
<video controls>
    <source src="./images/Material/material.webm" type="video/webm"/>
</video>
It's important to remember that the apparence of the entity is determined by the interplay between the light source and the material properties of the entity.
Above I used a white light with all paramters set to $$1.0$$ so it does not contribute in any meaningful manner.
I encourage you to play around with different parameter settings to get a feel for how they affect the scene.

[code](https://github.com/AdamSturge/Engine/tree/blog_material)

{% include links.html %}
