---
title: Force Accumulator
summary : "Breaking force related code out of Scene"
sidebar: mydoc_sidebar
permalink: mydoc_force_accumulator.html
folder: mydoc
toc : true
---

## Overview
If we are going to be adding move forces to our scene then we should encapsulate the force related code in it's own class. 
Otherwise a scene will be bloated with code that isn't really its responsibility. 
To that end we are going to introduce the *NetForceAccumulator* class, which as the name implies will accumulate all the forces acting on an entity.

## NetForceAccumulator
```c++
class NetForceAccumulator
{
    private:
        std::vector<ConstantForceGenerator> m_constant_forces;
        GravityForceGenerator m_gravity_force;
        bool gravity_on;

    public:

        /**
            Constructs a NetForceAccumulator
        **/
        NetForceAccumulator();

        /**
            Adds a new constant force to the simuluation
            @param a acceleration vector of the new constant force
        **/
        void AddConstantForce(Vector3Gf a);

        /**
            Enables/Disables universial gravitation between entities
            @param enable true to turn on gravtational attraction, false to turn off
        **/
        void EnableGravity(bool enable);

        /**
            Computes the net force acting on an entity in the sumulation via accumulation
            @param entity_ptrs list of all entities in the simulation
            @param entity_ptr entity whom the net force is acting on
            @param force vector that will be modified to contain the net force acting on the supplied entity
        **/
        void ComputeNetForce(
            const std::vector<std::shared_ptr<PhysicsEntity>> &entity_ptrs,
            const std::shared_ptr<PhysicsEntity> entity_ptr,
            Vector3Gf &force) const;

        /**
            Computes the net force jacobian for the supplied entity
            @param entity_ptrs list of all entities in the simulation
            @param entity_ptr entity whom the net force is acting on
            @param dFdx matrix that will be modified to contain the net force jacobian with respect to position
            @param dFdx matrix that will be modified to contain the net force jacobian with respect to velocity
        **/
        void ComputeNetForceJacobian(
            const std::vector<std::shared_ptr<PhysicsEntity>> &entity_ptrs,
            const std::shared_ptr<PhysicsEntity> entity_ptr,
            Eigen::Matrix<GLfloat,3,3> &dFdx,
            Eigen::Matrix<GLfloat,3,3> &dFdv) const;


};
```
```c++
NetForceAccumulator::NetForceAccumulator()
{

}

void NetForceAccumulator::AddConstantForce(Vector3Gf a)
{
    m_constant_forces.push_back(ConstantForceGenerator(a));
}

void NetForceAccumulator::EnableGravity(bool enable)
{
    gravity_on = enable;
}

void NetForceAccumulator::ComputeNetForce(
    const std::vector<std::shared_ptr<PhysicsEntity>> &entity_ptrs,
    const std::shared_ptr<PhysicsEntity> entity_ptr,
    Vector3Gf &force) const
{
    for(ConstantForceGenerator constant_force : m_constant_forces)
    {
        constant_force.AccumulateForce(entity_ptr,force);
    }

    if(gravity_on)
    {
        for(std::shared_ptr<PhysicsEntity> other_entity_ptr : entity_ptrs)
        {
            if(other_entity_ptr != entity_ptr)
            {
                m_gravity_force.AccumulateForce(entity_ptr,other_entity_ptr,force);
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
    if(gravity_on)
    {
        for(std::shared_ptr<PhysicsEntity> other_entity_ptr : entity_ptrs)
        {
            if(other_entity_ptr != entity_ptr)
            {
                m_gravity_force.AccumulatedFdx(entity_ptr,other_entity_ptr,dFdx);
            }
        }
    }
}
```

## TimeIntegrator
Since we only need the ability to compute forces to time integrate forward we can remove Scene from the dependency list for *TimeIntegrator* and replace it with *NetForceAccumulator*
This allows us to remove that ugly forward declaration we introduced in [Updated TimeIntegrator](mydoc_updated_time_integrator.html).
Naturally this change will require changing all the concrete implementations of *TimeIntegrator*, which you can grab from github at the bottom of this article. 
```c++
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
            @param net_force_accumulator accumulates the net force acting on entity_ptr
            @param entity_ptrs list of all the PhysicsEntities in the scene
            @param entity_ptr pointer to the entity being updated
         **/
         void StepForward(
            const NetForceAccumulator& net_force_accumulator,
            const std::vector<std::shared_ptr<PhysicsEntity>> &entity_ptrs,
            const std::shared_ptr<PhysicsEntity> entity_ptr);

        /**
            Returns the current step size for the integrator
        **/
        GLfloat GetStepSize();

    private:
        /**
            Private implementation details for solver.
            @param net_force_accumulator accumulates the net force acting on entity_ptr
            @param entity_ptrs list of all the PhysicsEntities in the scene
            @param entity_ptr pointer to the entity being updated
        **/
         virtual void Solve(
            const NetForceAccumulator& net_force_accumulator,
            const std::vector<std::shared_ptr<PhysicsEntity>> &entity_ptrs,
            const std::shared_ptr<PhysicsEntity> entity_ptr) = 0;
};
```

## Scene
All force related code in *Scene* is replaced with a NetForceAccumulator member variable. 
The only method that's changed as opposed to being flat out removed is *StepPhysics*, which now passes forward the *NetForceAccumulator* m_net_force_accumulator.
```c++
void Scene::StepPhysics()
{
    for(std::shared_ptr<PhysicsEntity> entity_ptr : m_physics_entity_ptrs)
    {
        m_time_integrator->StepForward(m_net_force_accumulator,m_physics_entity_ptrs,entity_ptr);
    }

    for(std::shared_ptr<PhysicsEntity> entity_ptr : m_physics_entity_ptrs)
    {
        entity_ptr->UpdateFromBuffers();
    }

};
```

## Results
no flashy results this time, just cleaner [code](https://github.com/AdamSturge/Engine/tree/blog_force_accumulator). 
{% include links.html %}
