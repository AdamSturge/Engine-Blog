---
title: Mesh
summary : "Define a model"
sidebar: mydoc_sidebar
permalink: mydoc_model.html
folder: mydoc
toc : true
---

## Overview
Last time we created a mesh, now we'll wrap our mesh class up in a **model** class.

## Model
A model for us will contain information about the entity the mesh represents. Properties such as color, reflectivity, etc are examples of things we could store in a model. However for the time being we're going to put all those aside and just store the **model matrix**. The model matrix handles transforming an entity from the **local coordinate sytem** where its vertex positions are defined into the **world coordinate system** that is shared by all entities in the scene. If you need a refresher check out this [tutorial](https://learnopengl.com/#!Getting-started/Coordinate-Systems) 

```c++
#include <mesh.h>
#include <Eigen/Core>
#include <vector3G.h>

#ifndef MODEL
#define MODEL
/**
    \brief Defines a model used for rendering

    Model is the base class for all entities that are rendered in a scene. It stores properites such as the mesh, material properties, textures, etc. It also holds the <b>model matrix</b> which handles transforming the mesh from local coordinates to world coordinates. See [Here](https://learnopengl.com/#!Getting-started/Coordinate-Systems) for more details.
**/
class Model {
   public:
        /**
            @return The model matrix
        **/
        Eigen::Matrix<GLfloat,4,4> GetModelMatrix();
       
        /**
            Sets the model matrix
            @param model Matrix to be set as the model matrix
        **/ 
        void SetModelMatrix(Eigen::Matrix<GLfloat,4,4> model);

        /**
            @return Copy of the mesh used by this model
        **/
        Mesh GetMesh();
        
     protected:
        /**
            The mesh for this model
        **/
        Mesh m_mesh; 
        /**
            The model matrix. This handles transforming from local model coordinates to world coordinates
        **/
        Eigen::Matrix<GLfloat,4,4> m_model_matrix;  
        
        /**
            Builds a Model with an empty mesh and identity model matrix
        **/
        Model();
    
};
#endif
````

```c++
#include <model.h>

Model::Model()
{
    m_model_matrix.setIdentity();
}

Eigen::Matrix<GLfloat, 4, 4> Model::GetModelMatrix()
{
    return m_model_matrix;
};

void Model::SetModelMatrix(Eigen::Matrix<GLfloat,4,4> m)
{
    m_model_matrix = m;
};

Mesh Model::GetMesh()
{
    return m_mesh;
}
````
{% include links.html %}
