---
title: Hooke's Law
summary : "A simple elastic force"
sidebar: mydoc_sidebar
permalink: mydoc_hookes_law.html
folder: mydoc
toc : true
---

## Overview
In this article we will implement a simple linear spring. 

## Springs
A linear spring is a spring with 2 entities on either end. 
It has a natural rest length $$l_{0}$$ that it desires to be in at all times. 
This means if the spring is deformed along the axis of the rest length then a restoring force will be generated to move the endpoints back into position. 
Each spring also has a stiffness, characterized by a real number $$k$$.

<img src="./images/Hookes Law/linear spring.jpg" />

## Hooke's Law
Hooke's law is the rule that governs the restoration force. 
Named after [Robert Hooke](https://en.wikipedia.org/wiki/Robert_Hooke), who famously disliked Issac Newton. 
\\[
\vec{F}(\vec{x}\_{1},\vec{x}\_{2}) = k(l - l_{0})\hat{n}
\\]
Where $$k$$ is the stiffness coefficent,
$$l_{0}$$ is the rest length, 
$$l = ||\vec{x}_{2} - \vec{x}_{1}||$$
and as always $$\hat{n} = \frac{\vec{x}_{2} - \vec{x}_{1}}{||\vec{x}_{2} - \vec{x}_{1}||}$$

### Jacobian
The Jacobians for Hooke's law is
\\[
\frac{\partial \vec{F}}{\partial \vec{x}} = -k(\hat{n}\hat{n}^{T} - \frac{(l - l_{0})}{l}(I - \hat{n}\hat{n}^{T}))
\\]
\\[
\frac{\partial \vec{F}}{\partial \vec{v}} = 0
\\]

## Implementation

### SpringForceGenerator
```c++
#ifndef SPRING_FORCE_H
#define SPRING_FORCE_H

#include <vector3G.h>
#include <memory>
#include <Eigen/Core>
#include <physics_entity.h>
#include <spring.h>

/**
    \brief Computes the spring force resulting from 2 entities being connected by a spring
**/
class SpringForceGenerator
{
    public :
        /**
            Constructs a SpringForceGenerator
        **/
        SpringForceGenerator();

        /**
            Adds a spring force due to 2 entities being connected by a spring
            @param k stiffness coefficent of spring
            @param l0 rest length of spring
            @param entity1_ptr entity on one end of the spring. The force will be computed such that it acts on this entity.
            @param entity2_ptr entity on the other end of the spring. The negative of the computed force acts on this entity.
            @param F Force vector that will accumulate the constant force represented by this instance
        **/
        void AccumulateForce(
            const GLfloat k,
            const GLfloat l0,
            const std::shared_ptr<PhysicsEntity> entity1_ptr,
            const std::shared_ptr<PhysicsEntity> entity2_ptr,
            Vector3Gf &F) const;

        /**
            Adds the position jacobian
            @param k stiffness coefficent of spring
            @param l0 rest length of spring
            @param entity1_ptr entity on one end of the spring. The force will be computed such that it acts on this entity.
            @param entity2_ptr entity on the other end of the spring. The negative of the computed force acts on this entity.
            @param dF Matrix that will accumulate the derivative of force with respect to position
        **/
        void AccumulatedFdx(
            const GLfloat k,
            const GLfloat l0,
            const std::shared_ptr<PhysicsEntity> entity1_ptr,
            const std::shared_ptr<PhysicsEntity> entity2_ptr,
            Eigen::Matrix<GLfloat,3,3> &dF) const;



};

#endif
```
```c++
#include <spring_force.h>

SpringForceGenerator::SpringForceGenerator(){};

void SpringForceGenerator::AccumulateForce(
    const GLfloat k,
    const GLfloat l0,
    const std::shared_ptr<PhysicsEntity> entity1_ptr,
    const std::shared_ptr<PhysicsEntity> entity2_ptr,
    Vector3Gf &F) const
{
    Vector3Gf x1 = entity1_ptr->GetPosition();
    Vector3Gf x2 = entity2_ptr->GetPosition();

    Vector3Gf n = x2 - x1;
    GLfloat l = n.norm();
    n = n/l;

    F += k*(l - l0)*n;
}

void SpringForceGenerator::AccumulatedFdx(
    const GLfloat k,
    const GLfloat l0,
    const std::shared_ptr<PhysicsEntity> entity1_ptr,
    const std::shared_ptr<PhysicsEntity> entity2_ptr,
    Eigen::Matrix<GLfloat,3,3> &dF) const
{
    Vector3Gf x1 = entity1_ptr->GetPosition();
    Vector3Gf x2 = entity2_ptr->GetPosition();

    Vector3Gf n = x2 - x1;
    GLfloat l = n.norm();
    n = n/l;

    Eigen::Matrix<GLfloat,3,3> N = n*n.transpose();
    Eigen::Matrix<GLfloat,3,3> I = Eigen::Matrix<GLfloat,3,3>::Identity();

    dF += -k*(N + ((l - l0)/l)*(I - N));
}
```
### Spring
Spring is a struct used to represent a spring connecting 2 PhysicsEntities
```c++
#ifndef SPRING_H
#define SPRING_H

#include <physics_entity.h>
#include <memory>
#include <atomic>
/**
    \brief data structure that represents a spring
**/
struct Spring
{
    private:
        static std::atomic<GLint> next_id;

    public :
        GLint id; // unique id for this spring
        GLfloat k; // stiffness coefficent
        GLfloat l0; // rest length
        std::shared_ptr<PhysicsEntity> entity1_ptr;
        std::shared_ptr<PhysicsEntity> entity2_ptr;

    /**
        Constructs a spring with default stiffness and rest length, and no entites on its endpoints
    **/
    Spring();

    /**
        Constucts a spring using the provided parameters
        @param k stiffness coefficent of the spring
        @param l0 rest length of the spring
        @param entity1_ptr pointer to entity on one end of the spring
        @param entity2_ptr pointer to entity on other end of the spring
    **/
    Spring(GLfloat k, GLfloat l0, std::shared_ptr<PhysicsEntity> entity1_ptr, std::shared_ptr<PhysicsEntity> entity2_ptr);
};
#endif
```
Each spring has a unique id that makes referencing it easier. 
To generate these ids we make use of the [atomic](http://en.cppreference.com/w/cpp/atomic/atomic) c++ type. 
Atomic ensures that if 2 processes are reading and writing from the same memory location what happens is well-defined. 
While this isn't important for now it might become important if we ever multi-thread our engine.

```c++
#include <spring.h>

std::atomic<GLint> Spring::next_id(0);

Spring::Spring()
{
    id = Spring::next_id++;
};

Spring::Spring(GLfloat k, GLfloat l0, std::shared_ptr<PhysicsEntity> entity1_ptr, std::shared_ptr<PhysicsEntity> entity2_ptr) : k(k), l0(l0), entity1_ptr(entity1_ptr), entity2_ptr(entity2_ptr)
{
    id = Spring::next_id++;
}
```

### PhysicsEntity
We've made due without it up to this point but now we are going to introduce a unique id for each PhyscisEntity in the Scene. 
I won't write the code here as it's the same code we used to generate spring ids.

### NetForceAccumulator
We'll we adding maps that map spring ids to springs, and entities to springs they are in.
```c++
class NetForceAccumulator
{
    private:
        // Same as before

        SpringForceGenerator m_spring_force;
        std::unordered_map<GLint, Spring> m_springs; // maps spring id to spring
        std::unordered_map<GLint,std::vector<GLint>> m_entity_spring_map; // maps physics id to springs it is a part of (by id)

    public :
        // Same as before

        /**
            Adds a spring to the force simulation
            @param spring spring to be added
        **/
        void AddSpring(Spring spring);

        // Same as before

```

```c++
void NetForceAccumulator::AddSpring(Spring spring)
{
    m_springs[spring.id] = spring;
    m_entity_spring_map[spring.entity1_ptr->GetId()].push_back(spring.id);
    m_entity_spring_map[spring.entity2_ptr->GetId()].push_back(spring.id);
}

void NetForceAccumulator::ComputeNetForce(
    const std::vector<std::shared_ptr<PhysicsEntity>> &entity_ptrs,
    const std::shared_ptr<PhysicsEntity> entity_ptr,
    Vector3Gf &force) const
{
    // Same as before

    GLint pid = entity_ptr->GetId();
    // Have to use find() instead of [] since we declared the method as const
    std::unordered_map<GLint,std::vector<GLint>>::const_iterator spring_itr = m_entity_spring_map.find(pid);
    if(spring_itr != m_entity_spring_map.end())
    {
        std::vector<GLint> spring_ids = spring_itr->second;
        for(GLint spring_id : spring_ids)
        {
            Spring spring = m_springs.find(spring_id)->second;
            if(spring.entity1_ptr == entity_ptr)
            {
                m_spring_force.AccumulateForce(spring.k,spring.l0,spring.entity1_ptr,spring.entity2_ptr,force);
            }
            else if(spring.entity2_ptr == entity_ptr)
            {
                // swap the order of entity1_ptr and entity2_ptr to produce the negative of the force
                m_spring_force.AccumulateForce(spring.k,spring.l0,spring.entity2_ptr,spring.entity1_ptr,force);
            }
            else
            {
                // We done goofed. 
                std::cout << "Computing a spring force on an entity not in this spring. PID : " << pid << " SpringID : " << spring_id << std::endl;
            }
        }
    }
}

void NetForceAccumulator::ComputeNetForceJacobian(
            const std::vector<std::shared_ptr<PhysicsEntity>> &entity_ptrs,
            const std::shared_ptr<PhysicsEntity> entity_ptr,
            Eigen::Matrix<GLfloat,3,3> &dFdx,
            Eigen::Matrix<GLfloat,3,3> &dFdv) const
{   
    GLint pid = entity_ptr->GetId();
    // Have to use find() instead of [] since we declared the method as const
    std::unordered_map<GLint,std::vector<GLint>>::const_iterator spring_itr = m_entity_spring_map.find(pid);
    if(spring_itr != m_entity_spring_map.end())
    {
        std::vector<GLint> spring_ids = spring_itr->second;
        for(GLint spring_id : spring_ids)
        {
            Spring spring = m_springs.find(spring_id)->second;
            m_spring_force.AccumulatedFdx(spring.k,spring.l0,spring.entity1_ptr,spring.entity2_ptr,dFdx);
        }
    }
}
```
This code looks a little gnarly but at high level it's not doing a lot. 
In *ComputeNetForce* there's some new code to first check if the provided physics entity is in any springs.
If it finds some it pulls the springs out of the spring map and computes the resulting spring force.
One thing that we have to be careful of is the sign of the spring force.
The entity on one end of the spring should experience a force equal but oppsite the entity on the other end of the spring.
This should be clear because if both entites experienced the same force then the whole spring system would move in the direction of the force,
which clearly is not what happens in realtiy. 

### Scene
We need to add a method that lets us add *Spring* instances to the *Scene*. This method simply passes the spring down to the *NetForceAccumulator* *AddSpring* method

## Results
The resulting video was made using *SymplecticEuler* as the time integrator
<video controls>
    <source src="./images/Hookes Law/hookes law symplectic euler.webm" type="video/webm" />
</video>

Interestingly if we use any of the other symplectic integrators we implemented the spring system tends to blow up.
Here's a video of the same system using *Verlet* integration

<video controls>
    <source src="./images/Hookes Law/hookes law verlet.webm" type="video/webm" />
</video>

Notice how the appex of the movement of the spheres get closer/further apart over time? 
The Energy in a spring is directly proportional to the amount of compression it stores. 
The fact the spheres are compressing/stretching the spring more means they are slowly adding energy to the system. 
Recall that symplectic methods only **nearly** conserve energy.
It turns out in this case that the slight damping affect that Symplectic Euler has keeps the system under control for longer.


[code](https://github.com/AdamSturge/Engine/tree/blog_hookes_law)

{% include links.html %}
