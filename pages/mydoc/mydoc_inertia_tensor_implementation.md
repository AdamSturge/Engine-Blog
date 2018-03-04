---
title: Inertia Tensor Implementation
summary : "Implementing inertia tensors"
sidebar: mydoc_sidebar
permalink: mydoc_inertia_tensor_impl.html
folder: mydoc
toc : true
---

## Overview
In our [last article](/mydoc_inertia_tensor_theory.html) we went over some of the theory of inertia tensors. 
Now we'll go over the implementation of them for the rectangular prism

## Model Matrix
Near the end of our previous discussion we talked about coordinate systems. 
Up until now the model matrix, which handles converting from model coordinates to world coordinates, as been a graphics component. 
However now we need it for a physics calculation. 
This means we'll need to break out the model matrix into a new class *LocalCoorindateEntity*. 
 
``` c++
fndef LOCAL_COORDIANTE_ENTITY_H
#define LOCAL_COORDIANTE_ENTITY_H

#include <Eigen/Dense>
#include "vector3G.h"
/*
 * \brief Defines an entity that tracks how to transform from it's own local coordiante sytem to world coordinates
 */
class LocalCoordinateEntity{

  public:

        LocalCoordinateEntity();

	~LocalCoordinateEntity();

	/**
            @return The model matrix
        **/
        Eigen::Matrix<GLfloat,4,4> GetModelMatrix();
       
        /**
            Sets the model matrix
            @param model Matrix to be set as the model matrix
        **/ 
        void SetModelMatrix(Eigen::Matrix<GLfloat,4,4> model);


  protected:
	/**
            The model matrix. This handles transforming from local model coordinates to world coordinates
        **/
    	Eigen::Matrix<GLfloat,4,4> m_model_matrix; 

        virtual void OnModelMatrixUpdate() = 0;	


};
#endif
```
Now *Model* and *PhysicsEntity* inherit from *LocalCoorindateEntity*
```c++
class Model : virtual public LocalCoordinateEntity
```

```c++
class PhysicsEntity : virtual public LocalCoordinateEntity
```

Note that the inhertience is virtual so that we avoid the [diamond problem](https://medium.freecodecamp.org/multiple-inheritance-in-c-and-the-diamond-problem-7c12a9ddbbec)
if a class inherits from both Model and PhysicsEntity.

{% include warning.html content = "I'm really starting to see why engine developers like the [Entity-Component](https://en.wikipedia.org/wiki/Entity%E2%80%93component%E2%80%93system) design pattern. 
These hierachical structures can get messy" %}

## Inertia Matrix
The most natural place to keep the inertia matrix is in *PhysicsEntity*. 
However we won't be storing $$J$$ but $$J^{-1}$$.
If you go back to our discussion of time integrators the general approach was to comptue the net force $$\vec{F}$$ and then compute $$\vec{a} = \frac{\vec{F}}{m}$$.
Since the way $$m$$ is used is always as $$m^{-1}$$ commenical engines all store inverse mass under the hood. 
In this way they can avoid computing the an inverse every physics cycle. 
This was an optimization I never bothered with in the case of linear forces since inverting a float isn't **that** bad, and this blog is meant to be educational. 
However inverting a matrix every cycle is really expensive. 
So instead we'll store $$J^{-1}$$ 

```c++
/*
* Inverse of interial tensor for this entity in world coordinates
*/
Eigen::Matrix<GLfloat,3,3> m_inverse_inertia_tensor_world;

/**
 * Inverse of interial tensor for this entity in local coordinates
 */
Eigen::Matrix<GLfloat,3,3> m_inverse_inertia_tensor_local;
```
*m_inverse_inertia_tensor_local* is computed once in a local coordinate frame that is set up to make it a simple calculation,
frequently a diagonal matrix.
*m_inverse_inertia_tensor_world* is recomputed every cycle based on the current value of the model matrix.

The only new method (aside from getters/setters) is
```c++
csEntity::OnModelMatrixUpdate()
{
  m_inverse_inertia_tensor_world = m_model_matrix.block(0,0,3,3)*m_inverse_inertia_tensor_local;
}
```
and that's it for general purpose code. 

## Computing local inertial tensors
In our case we are only concerned about a known regular solid. 
These have well known closed form inertial tensors which we will use for this tutorial.
However in general our engine should support arbitrary polygons. 
There's a very cool way that this is done which I'll put off until a later article. 
But as a teaser it involves using [Divergence Theorem](https://en.wikipedia.org/wiki/Green%27s_theorem) and [Green's Theorem](https://en.wikipedia.org/wiki/Green%27s_theorem) to turn 3 dimensional integrals into 1 dimensional integrals. 

[This page](https://en.wikipedia.org/wiki/List_of_moments_of_inertia#List_of_3D_inertia_tensors) has a list of inertia tensors for common rigid bodies.
Of interest to us is the rectangular prism
\\[
\begin{bmatrix}
\frac{1}{12}m(h^{2} + w^{2}) & 0 & 0 \\\
0 & \frac{1}{12}m(l^{2} + w^{2}) & 0 \\\
0 & 0 & \frac{1}{12}m(l^{2} + h^{2}) \\\
\end{bmatrix}
\\]

```c++
RectangularPrism::RectangularPrism(GLuint length, GLuint width, GLuint height, Vector3Gf position, Vector3Gf velocity, GLfloat mass, Quaternion orientation, Vector3Gf angular_velocity, Material material) : Model(), PhysicsEntity(position,velocity,mass,orientation,angular_velocity)
{
    m_length = length; 
    m_width  = width; 
    m_height = height; 

    m_model_matrix.col(3) << position(0), position(1), position(2), 1.0f; 
    m_model_matrix.block(0,0,3,3) = orientation.toRotationMatrix();
    GenerateMesh();

    m_material = material;

    Eigen::Matrix<GLfloat,3,3> I;
    I.setZero();
    I(0,0) = width*width + height*height;
    I(1,1) = width*width + length*length;
    I(2,2) = height*height + length*length;
    I = (1.0f/12.0f)*mass*I;
    SetLocalInertiaTensor(I);
}
```

## Conclusion
And there we have it. 
Unfortunatley we can't really test our new code until we start computing *torques* and write a solver for Euler's equation of motion. So no fancy video at the end of this series of posts.


{% include links.html %}
