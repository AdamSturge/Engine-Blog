---
title: Updated TimeIntegrator
summary : "Updating TimeIntegrator for more advanced integration schemes"
sidebar: mydoc_sidebar
permalink: mydoc_updated_time_integrator.html
folder: mydoc
toc : true
---

## Overview
So this is a little embarrasing but back in [Explicit Euler](mydoc_explicit_euler.html) when we created the TimeIntegrator class I tried to make it as specialized as possible by having it take as inputs the physical quantities at the start of the time step.
That way it would not require any knowledge of the engine at large. 
Turns out that was more limiting than I anticipated. 
So before we move onto more advanced solvers we are going to have to revisit TimeIntegrator. 

## TimeIntegrator 2.0
```c++
#ifndef TIME_INTEGRATOR_H
#define TIME_INTEGRATOR_H
#include <Eigen/Core>
#include <GL/glew.h>
#include <vector3G.h>
#include <scene.h>

class Scene; // Forward definition of Scene

/**
    \brief An abstact base class for all grid based numerical ODE solvers for Newton's laws
**/
class TimeIntegrator
{
    protected:
        GLfloat m_dt;

    public:
         virtual ~TimeIntegrator(){};
        
        /**
            Public interface for all solvers. 
            Computes position and velocity updates by solving Newton's equations of motion.
            Updates entity_ptr buffers with next predicted position and velocity
            @param scene scene containing entities to be integrated forward
            @param entity_ptr pointer to the entity being updated
         **/
         void StepForward(
            const Scene& scene,
            const std::shared_ptr<PhysicsEntity> entity_ptr);

    private:
        /**
            Private implementation details for solver.
            @param scene scene containing entities to be integrated forward
            @param entity_ptr pointer to the entity being updated
        **/
         virtual void Solve(
            const Scene& scene,
            const std::shared_ptr<PhysicsEntity> entity_ptr) = 0;
};
#endif
```

Looks pretty much like it did before except instead we are passing in the Scene and the whole PhysicsEntity. 
As we go forward through these articles it will be come more apparent why we need to do this.
Also note the [foward declaration](https://en.wikipedia.org/wiki/Forward_declaration) of Scene at the top of TimeIntegrator. 
That is requried since TimeIntegrator uses Scene and Scene uses TimeIntegrator. 
This is a red flag of poor design as it represents a contract that the compiler has to trust the author to satisfy, and was one of the reasons I didn't define TimeIntegrator this way in the first place.
However its not too bad for now and we can fix that in a later article. 

The cpp file has barely changed
```c++
#include <time_integrator.h>
#include <assert.h>

void TimeIntegrator::StepForward(
            const Scene& scene,
            const std::shared_ptr<PhysicsEntity> entity_ptr)

{
    assert(entity_ptr->GetMass() > 0.0f);

    Solve(scene,entity_ptr);
}
```

## Explicit Euler 2.0
Nothing going on here but updating ExplicitEuler to match the new interface

```c++
#include <time_integrator.h>
#include <Eigen/Core>
#include <vector3G.h>

#ifndef EXPLICIT_EULER
#define EXPLICIT_EULER
/**
    \brief Implementation of Explicit Euler method for solving Newton's equations of motion
**/
class ExplicitEuler : public TimeIntegrator
{
    public :
        /**
            Constructs an instance of ExlplictEuler with a default grid size
        **/
        ExplicitEuler();

        /**
            Constructs an instance of ExplicitEuler with the provided grid size
            @param dt difference between two points on the grid
        **/
        ExplicitEuler(GLfloat dt);

    private:

        /**
            Solves Newton's equations of motion.
            Updates entity_ptr buffers with next predicted position and velocity
            @param scene scene containing entities to be integrated forward
            @param entity_ptr pointer to entity to be integrated forward
        **/
        void Solve(
            const Scene& scene,
            const std::shared_ptr<PhysicsEntity> entity_ptr);

};
#endif
```
```c++
#include <explicit_euler.h>
#include <assert.h>
#include <iostream>

ExplicitEuler::ExplicitEuler()
{
    m_dt = 0.001;
};

ExplicitEuler::ExplicitEuler(GLfloat dt)
{
    m_dt = dt;
};

void ExplicitEuler::Solve(const Scene& scene,const std::shared_ptr<PhysicsEntity> entity_ptr) 
{   
    const Vector3Gf xi = entity_ptr->GetPosition();
    const Vector3Gf vi = entity_ptr->GetVelocity();
    const GLfloat mass = entity_ptr->GetMass();
   
    Vector3Gf F;
    F.setZero();
    scene.ComputeNetForce(entity_ptr,F);

    Vector3Gf xf = xi + m_dt*vi;
    Vector3Gf vf = vi + m_dt*(1/mass)*F; 

    entity_ptr->SetNextPosition(xf);
    entity_ptr->SetNextVelocity(vf);  
};

```
You'll notice that *Solve()* now updates the position and velocity buffers for the entity, something which used to be the responsiblity of Scene.

## Scene 2.0
Some minor changes in Scene accompany this change in interface for TimeInterator.
We have to add a forward declaration to TimeIntegrator and we have moved ComputeNetForce from private to public.
```c++
#ifndef SCENE
#define SCENE

#include <physics_entity.h>
#include <model.h>
#include <memory>
#include <vector3G.h>
#include <time_integrator.h>
#include <constant_force.h>
#include <shader.h>
#include <camera.h>
#include <GLFW/glfw3.h>
#include <gravity_force.h>

class TimeIntegrator; // Forward declaration of TimeIntegrator. 
```

```c++
/**
    \brief A scene in the engine

    A scene contains (pointers to) a number of PhysicsEntity objects to simululate and a number of Model objects to render.
**/
class Scene
{
    public :

        /**
            Computes the net force acting on an entity
            @param entity_ptr pointer to the entity the forces are acting on
            @param force force vector that will be modified to contain the net force
        **/
       void ComputeNetForce(const std::shared_ptr<PhysicsEntity> entity_ptr, Vector3Gf &force) const
};
#endif
```

The only update to the cpp file is in *StepPhysics()*
```c++
void Scene::StepPhysics()
{
    for(std::shared_ptr<PhysicsEntity> entity_ptr : m_physics_entity_ptrs)
    {
        m_time_integrator->StepForward(*this,entity_ptr);
    }

    for(std::shared_ptr<PhysicsEntity> entity_ptr : m_physics_entity_ptrs)
    {
        entity_ptr->UpdateFromBuffers();
    }

};
```

## Code
With that we are ready to take on some more advanced solvers. 

Full [code](https://github.com/AdamSturge/Engine/tree/blog_updated_time_integrator)

{% include links.html %}
