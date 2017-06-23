---
title: Lets get physical
summary : "Add physical properties to our UV sphere"
sidebar: mydoc_sidebar
permalink: mydoc_lets_get_physical.html
folder: mydoc
toc : true
---

## Overview
So far we haven't spoken about any physics, which is going to be one of the main focuses of our Engine. Lets change that. In this article we'll be adding some [physical observables](https://en.wikipedia.org/wiki/Observable) to our sphere. 

## Physics Entity
For us a physics entity will be an object with a position and velocity as measured from world coordinates, as well as a mass. We won't be dealing with rotational dynamics at this time to keep things simple. In its simplist form this would look something like this.

```c++
#include <Eigen/Core>
#include <vector3G.h>

#ifndef PHYSICS_ENTITY
#define PHYSICS_ENTITY
/**
    \brief An entity subject to the laws of physics

    PhysicsEntity is a base class for all objects in a scene that respond to forces and other physical phenomenon  
**/
class PhysicsEntity{
   public:
        /**
            @return Spatial position for this entity
        **/
        Vector3Gf GetPosition();

        /**
            @return Spatial velocity for this entity
        **/
        Vector3Gf GetVelocity();

        /**
            @return Mass of this entity
        **/
        GLfloat GetMass();
 
     protected:     
        /**
            The position of this entity
        **/
        Vector3Gf m_position;
         
        /**
            The velocity of this entity
        **/
        Vector3Gf m_velocity;
        
        /**
            The mass of this entity
        **/
        GLfloat m_mass; // stores the mass of this game entity

        /**
            Creates an instance of PhysicsEntity centered at the origin, zero velocity, and unit mass
        **/
        PhysicsEntity();
    
};
#endif
```
You may notice that the member variables are protected instead of private. That's because the envisioned use for PhysicsEntity is as a base class that other game entities can extend in a manner simlar to the [Model](mydoc_model.html) class. 
Luckily (or unluckily if you ask some) C++ supports [multiple inhertitance](http://www.cprogramming.com/tutorial/multiple_inheritance.html) so our UV sphere can extend both. 

{% include warning.html content="
If I'm going to be honest I'm no C++ expert. I do know that multiple inhertiance is a contentious topic but in this instance it makes sense to me. Maybe in the future this will come back to bite me in the ass and I'll re-read this paragraph shaking my head at the fool I was.
But for now lets soldier on."
%}

As it stands our PhysicsEntity doesn't have any means to update it's member variables from outside. A static object isn't very interesting so we should change that. However for reasons that will become clear later we can't simply implement a setter to change the observables. What we need to do is store is the next predicted values seperately, and implement an update method that loads them from their buffers. Putting that together we get this

```c++
#include <mesh.h>
#include <Eigen/Core>
#include <vector3G.h>

#ifndef PHYSICS_ENTITY
#define PHYSICS_ENTITY
/**
    \brief An entity subject to the laws of physics

    PhysicsEntity is a base class for all objects in a scene that respond to forces and other physical phenomenon  
**/
class PhysicsEntity{
   public:
        /**
            @return Spatial position for this entity
        **/
        Vector3Gf GetPosition();

        /**
            Sets the next position buffer for this entity. This <b>DOES NOT</b> update the position until UpdateFromBuffers() is called.
            @param x Next position for this entity
        **/
        void SetNextPosition(Vector3Gf x);

        /**
            @return Spatial velocity for this entity
        **/
        Vector3Gf GetVelocity();

        /**
            Sets the next velocity buffer for the entity. This <b>DOES NOT</b> update the velocity until UpdateFromBuffers() is called.
            @param v Next velocity for this entity
        **/
        void SetNextVelocity(Vector3Gf v);

        /**
            @return Mass of this entity
        **/
        GLfloat GetMass();

        /**
            Updates internal state from buffers. This will load the next position and velocity from their corresponding buffers
        **/
        void UpdateFromBuffers();

     protected:     
        /**
            The position of this entity
        **/
        Vector3Gf m_position;
        /**
            The next predicted position of this entity. 
        **/
        Vector3Gf m_next_position_buffer;    
        /**
            The velocity of this entity
        **/
        Vector3Gf m_velocity;
        /**
            The next predicted velocity of this entity
        **/
        Vector3Gf m_next_velocity_buffer;
        /**
            The mass of this entity
        **/
        GLfloat m_mass; // stores the mass of this game entity

        /**
            Creates an instance of PhysicsEntity centered at the origin, zero velocity, and unit mass
        **/
        PhysicsEntity();

        /**
            Destructor for PhysicsEntity. Virtual because of the presense of virtual methods 
        **/
       virtual ~PhysicsEntity();

    private :
        /**
            Called after UpdateFromBuffers. Allows subclasses to perform class specific actions when physical observables update
        **/
        virtual void OnUpdateFromBuffers() {};
    
};
#endif
```
Most of that should be self explanatory. The *UpdateFromBuffers* method copies the new position and velocity into m_position and m_velocity. Then it calls *OnUpdateFromBuffers*. 
This second method allows subclasses to listen for the update event and provide custom actions specific to their own member variables. 
For example our UV sphere will want to update it's m_center variable to reflect it's new position in world coordinates. 

As for why the destructor for PhysicsEntity is virtual. Since we plan on using PhysicsEntity in a polymorphic manner, and it contains a virtual method, then in order for poitners to properly delete it's destructor [must be virtual](https://stackoverflow.com/questions/461203/when-to-use-virtual-destructors).

Here is the [cpp](https://github.com/AdamSturge/Engine/blob/blog_lets_get_physical/physics_entity.cpp) file for PhysicsEntity. And here is the updated UV Sphere class [sphere.h](https://github.com/AdamSturge/Engine/blob/blog_lets_get_physical/include/sphere.h) [sphere.cpp](https://github.com/AdamSturge/Engine/blob/blog_lets_get_physical/sphere.cpp)


{% include links.html %}
