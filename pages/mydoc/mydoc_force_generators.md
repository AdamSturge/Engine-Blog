---
title: Force generators
summary : "Adding a constant downward force to the scene"
sidebar: mydoc_sidebar
permalink: mydoc_force_generators.html
folder: mydoc
toc : true
---

## Overview
In this article we are going to introduce the concept of a **force generator**. 
Like the name implies a force generator computes a specific kind of force.
For this introductory article we will be keeping things simple with a constant force. 
Constant forces act in one fixed direction and do not vary in space or time.

## Forces
So far we've been dealing with forces in a handwavy maner without really talking about what they are.
In general a force is a 3D vector. 
\\[
\vec{F}(\vec{x}(t)) = m\frac{\partial^{2} \vec{x}}{\partial t^{2}}
\\]

You can think of a force as a [force field](https://en.wikipedia.org/wiki/Force_field_(physics)). 
A force field defines the force a particular object experiences as a function of it's position.
A classic example of this is the magnetic force field you see in any good intro science textbook. 

<img src="./images/Force Generator/field_lines.gif" />
<img src="./images/Force Generator/iron_filings.png" />

The arrows define the direction of the force at that point in space.
In much the same way that a ball floating on the ocean is carried by water currents physical objects are carried through space by force currents.

{%include note.html content="Technically what we have defined is a [conservative force](https://en.wikipedia.org/wiki/Conservative_force).
Conservative forces only depend on the position of the particle, not on time. It's important to clarify that the position itself is a function of time, so in a manner the force is as well.
Sometimes people write $$\vec{F}(t)$$ to indicate this implicit dependance on time. However a more accurate way to write it is $$\vec{F}(\vec{x}(t))$$.
It is possible to have forces that are directly dependant on time, but we won't have any use for them." %}

## Constant force
For a constant force there is no (implicit) time dependance.
Or put another way $$\frac{\partial^{2} \vec{x}}{\partial t^{2}}$$ is a constant vector. 
For example the downward acceleration due to gravity on the surface of the Earth is constant. 
(Technically it does vary a little between the equator to the poles but not enough to worry about in this context).
For Earth-like gravity $$\frac{\partial^{2} \vec{x}}{\partial t^{2}} \approx (0,-9.81,0)$$

## Net force
The **net force** acting on an object is the vector sum of all the forces acting on it. This leads us to the natural strategy of **force accumulation**. 
During a time step $$t_{i}$$ we compute the net force acting on each object by passing a single force vector through a series of force generators, each of which adds a value to the net force vector.
The net force vector can then by used by whatever time integration scheme you like to solve for the position and velocity updates.

## Constant force generator
Here is a constant force generator
```c++
#include <Eigen/Core>
#include <GL/glew.h>
#include <physics_entity.h>
#include <memory>
#include <vector3G.h>

#ifndef CONSTANT_FORCE_GENERATOR
#define CONSTANT_FORCE_GENERATOR
/**
    \brief A constant force throughout all space 
**/
class ConstantForceGenerator
{
    private:
        /** Spacially and temporarly uniform acceleration vector represeting constant force **/
        Vector3Gf m_accel;
    public:
        /**
            Builds a ConstantForceGenerator with a zero acceleration at all points in space.
        **/
        ConstantForceGenerator();

        /**
            Builds a ConstantForceGenerator with a given acceleration at all points in space
            @param accel acceleration vector for the force
        **/
        ConstantForceGenerator(Vector3Gf accel);

        /**
            Deconstructs a ConstantForceGenerator
        **/
        ~ConstantForceGenerator();

        /**
            Adds a constant force determined by the mass of the provided entity and the stored acceleration vector
            @param entity Entity whom the force will be applied against
            @param F Force vector that will accumulate the constant force represented by this instance
        **/
        void AccumulateForce(std::shared_ptr<PhysicsEntity> entity, Vector3Gf &F);

};
#endif
```
After everything we've talked about it should be apparent what each method does.

```c++
#include <constant_force.h>

ConstantForceGenerator::ConstantForceGenerator()
{
    m_accel = Vector3Gf(0.0f,0.0f,0.0f);
};


ConstantForceGenerator::ConstantForceGenerator(Vector3Gf accel)
{
    m_accel = accel;
};

ConstantForceGenerator::~ConstantForceGenerator(){};

void ConstantForceGenerator::AccumulateForce(std::shared_ptr<PhysicsEntity> entity, Vector3Gf &F)
{
    GLfloat mass = entity->GetMass();
    
    F += mass*m_accel;
    
};
```

## Scene
Now that we have our first force generator we need to put it into the scene. 
We'll add a new member variable to scene to store a ConstantForceGenerator.
We'll also add a new method that computes the net force acting on a PhysicsEntity.

```c++
class Scene
{
    private:    
        std::vector<std::shared_ptr<PhysicsEntity>> m_physics_entity_ptrs;
        std::vector<std::shared_ptr<Model>> m_model_ptrs;
        std::shared_ptr<TimeIntegrator> m_time_integrator;
        ConstantForceGenerator m_constant_force_generator;
           
       
        /**
            Computes the net force acting on an entity
            @param entity_ptr pointer to the entity the forces are acting on
            @param force force vector that will be modified to contain the net force
        **/
        void ComputeNetForce(const std::shared_ptr<PhysicsEntity> entity_ptr, Vector3Gf &force);
    public :
        /**
            Creates a Scene instance with default values for its TimeIntegrator and ForceGenerator members
        **/
        Scene();

        /**
            Creates a Scene instance with the provided TimeIntegrator and ForceGenerator members
            @param integrator TimeIngrator instance to handle time evolution of the PhysicsEntities in the scene
            @param cfg  ConstantForceGenerator used to generate a spatially and temporally uniform force
        **/
        Scene(std::shared_ptr<TimeIntegrator> integrator, ConstantForceGenerator cfg);

//The rest is the same as before
```

The following updates to scene.cpp complete our implementation

### Constructors
```c++
Scene::Scene() 
{  
    m_time_integrator = std::shared_ptr<TimeIntegrator>(new ExplicitEuler(0.01f)); 
    m_constant_force_generator = ConstantForceGenerator(Vector3Gf(0.0f,0.0f,0.0f)); 
}; 
 
Scene::Scene(std::shared_ptr<TimeIntegrator> integrator, ConstantForceGenerator cfg) 
{ 
    m_time_integrator = integrator; 
    m_constant_force_generator = cfg; 
}
```

### StepPhysics
```c++
void Scene::StepPhysics()
{
    for(std::shared_ptr<PhysicsEntity> entity_ptr : m_physics_entity_ptrs)
    {
        Vector3Gf xi = entity_ptr->GetPosition();
        Vector3Gf vi = entity_ptr->GetVelocity();
        GLfloat mass = entity_ptr->GetMass();

        Vector3Gf force;
        force.setZero();

        // This is new!
        ComputeNetForce(entity_ptr,force);

        Vector3Gf xf;
        Vector3Gf vf;

        m_time_integrator->StepForward(xi,vi,mass,force,xf,vf);

        entity_ptr->SetNextPosition(xf);
        entity_ptr->SetNextVelocity(vf);

    }

    for(std::shared_ptr<PhysicsEntity> entity_ptr : m_physics_entity_ptrs)
    {
        entity_ptr->UpdateFromBuffers();
    }

};
```
### ComputeNetForce
```c++
void Scene::ComputeNetForce(const std::shared_ptr<PhysicsEntity> entity_ptr, Vector3Gf &force)
{
    m_constant_force_generator.AccumulateForce(entity_ptr,force);
};

```

## Results
If you update main.cpp to utilize the new ConstantForceGenerator and give it a downward force vector of $$(0,-9.81,0)$$ you should see something like this.

<video controls>
    <source src="./images/Force Generator/gravity_sim.webm" type="video/webm" />
</video>

Here is the full [code](https://github.com/AdamSturge/Engine/tree/blog_force_generator).

And with that we've completed what I could call our first fully featured physics simulation.
We are computing forces and solving Newton's equations of motion on each frame. 
And we've done so in an extensible way. 
Adding new forces is as simple as creating new force generators, and adding new integration schemes merely requires creating a new class that inherits from TimeIntegrator. 
Congradulations!
{% include links.html %}
