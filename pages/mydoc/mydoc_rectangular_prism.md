---
title: Rectangular Prism
summary : "Adding a rechtangular prism primative to our simulations"
sidebar: mydoc_sidebar
permalink: mydoc_rectangular_prism.html
folder: mydoc
toc : true
---

## Overview
In the following series of articles we are going to augment our physics simulation with [rigid body](https://en.wikipedia.org/wiki/Rigid_body) dynamics.
In laymens terms a rigid body is an object whose deformation under pressure can be ignored.
Meaning $$2$$ points on or within its surface will always be the same distance apart.
Most objects we interact with in everyday life fall into this category.
The main difference between rigid body motion and what we've implemented so far is rigid bodies can rotate.


However before we get to any of that we need to address an issue. 
Namely that up until this point we've only been dealing with spheres, which are rotationally symetric. 
There's not much point in implementing rotational physics if our entities look the same under rotations. 
To that end we will put together a simple rectangular prism, ie.) a box.

## Rectangular prism
In general rectangular prisms are characterized by 3 side lengths and a position for their center. 
Since we'll be working in model coordinates we can assume the center to be the origin.
For our purposes length will be measured along the $$x$$ axis, width the $$y$$ axis, and height the $$z$$ axis.
```c++
#ifndef RECTANGULAR_PRISM_H
#define RECTANGULAR_PRISM_h
#include <model.h>
#include <physics_entity.h>
#include <vector3G.h>
#include <material.h>

class RectangularPrism : public Model, public PhysicsEntity
{
    private:
        GLuint m_length;
        GLuint m_width;
        GLuint m_height;

        void GenerateMesh();

    public:
        RectangularPrism(GLuint length, GLuint width, GLuint height, Vector3Gf postion, Vector3Gf velocity, GLfloat mass, Material material);

        /**
            Loads the next position and velocity values from their respective buffers
        **/
        void OnUpdateFromBuffers();

};
#endif
```

The only interesting method is *GenerateMesh*
```c++
include <rectangular_prism.h>
#include <iostream>
RectangularPrism::RectangularPrism(GLuint length, GLuint width, GLuint height, Vector3Gf position, Vector3Gf velocity, GLfloat mass, Material material) : Model(), PhysicsEntity(position,velocity,mass)
{
    m_length = length;
    m_width  = width;
    m_height = height;

    m_model_matrix.col(3) << position(0), position(1), position(2), 1.0f;
    GenerateMesh();

    m_material = material;
}

void RectangularPrism::GenerateMesh()
{
    List3df vertices(8,3);
    List3di faces(12,3);

    GLfloat l = (GLfloat)m_length/2.0f;
    GLfloat w = (GLfloat)m_width/2.0f;
    GLfloat h = (GLfloat)m_height/2.0f;

    // front face
    vertices.row(0) = Vector3Gf(-l,-w,h); // bottom left
    vertices.row(1) = Vector3Gf(-l, w,h); // top left
    vertices.row(2) = Vector3Gf( l,-w,h); // bottom right
    vertices.row(3) = Vector3Gf( l, w,h); // top right

    // back face
    vertices.row(4) = Vector3Gf(-l,-w,-h); // bottom left
    vertices.row(5) = Vector3Gf(-l, w,-h); // top left
    vertices.row(6) = Vector3Gf( l,-w,-h); // bottom right
    vertices.row(7) = Vector3Gf( l, w,-h); // top right

    // front face
    faces.row(0) = Vector3Gi(0,2,1);
    faces.row(1) = Vector3Gi(2,3,1);

    // back face
    faces.row(2) = Vector3Gi(6,4,7);
    faces.row(3) = Vector3Gi(4,5,7);

    // left face
    faces.row(4) = Vector3Gi(4,0,5);
    faces.row(5) = Vector3Gi(0,1,5);

    // right face
    faces.row(6) = Vector3Gi(2,6,3);
    faces.row(7) = Vector3Gi(6,7,3);

    // bottom face
    faces.row(8) = Vector3Gi(4,2,0);
    faces.row(9) = Vector3Gi(4,6,2);

    // top face
    faces.row(10) = Vector3Gi(3,7,1);
    faces.row(11) = Vector3Gi(7,5,1);

    m_mesh = Mesh(vertices,faces);
}

void RectangularPrism::OnUpdateFromBuffers()
{
    m_model_matrix.setIdentity();
    m_model_matrix.col(3)  << m_position(0), m_position(1), m_position(2), 1.0f;
}
```
Generate mesh works by first placing the $$8$$ vertices that represent the corners of the prism into the vertex list. 
From there it adds the $12$ faces by specifying vertex indices in counter clockwise order. 

You'll notice that in contrast to how we built our sphere mesh we reused vertices in multiple faces.
In addition to being more memory efficent this should result in smoother lighting on the surface.

## Results
<img src="./images/Rectangular Prism/cube.png" />

Here's the [code](https://github.com/AdamSturge/Engine/tree/blog_rectangular_prism)

{% include links.html %}
