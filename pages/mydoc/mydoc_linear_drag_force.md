---
title: Linear Drag
summary : "A simple velocity dependent drag force"
sidebar: mydoc_sidebar
permalink: mydoc_linear_drag_force.html
folder: mydoc
toc : true
---

## Overview
In this article we are going to introduce the first force to our arsenal that is velocity dependent. 
That force is going to be a drag force. 
Drag forces act to bleed away [mechanical energy](https://en.wikipedia.org/wiki/Mechanical_energy) from the system.
If you recall our defintion of conservative force from [force generators](mydoc_force_generators.html) forces that conserve mechanical energy only depend on position. 
So it is natural that our drag force will have a velocity component. 

## Drag
A drag force is way for us to model the effects of the environment on the objects in our simulation without modeling each effect individually.
For example when an object moves through the air some of its kenetic energy is converted to heat by air friction, or coverted to sound as it moves air out of its path. 
Instead of modeling these complicated interactions in detail it is often permittable to introduce a "drag force" that is encompases the net affect.

\\[
\vec{F}(\vec{v}) = -\beta m \vec{v}
\\]

Here $$\beta$$ is called the **drag coefficent**. It is a positive number that controls the strength of the drag effect. 
Notice that the negative sign out in front will cause the force to act in the direction opposite the velocity $$\vec{v}$$.
This is what will cause the entity to slow down over time. 
Because of the linear dependence of velocity this kind of drag is sometimes called **Linear Drag**. 
It's one of the simplest ways to model drag as it does not account for things like fluid density, cross sectional area, material properties, or any of the other myriad things that can affect drag.
One of the easier alterations one can make to this kind of drag force is to give each object its own $$\beta$$ value as opposed to using one global value.
Perhps I'll do a future article on a more complicated drag force.

{% include note.html content="The astute reader will notice that our drag force as written is not really a force. 
Forces have units of $$kg m/s^{2}$$ whereas we have above is $$kg m/s$$, which is technically a momentum. 
The easiest way to remedy is this to attach $$s^{-1}$$ units to $$\beta$$.
This is not very intellectually nourishing however and is really a kind of hack.
More physically motivated drag forces do not suffer from this problem"%}

### Jacobian
Since we want to support using implicit time integrators whenever we introduce a new force we'll want to also compute it's jacobian.
Luckily for us this drag force is simple. 
\\[
\frac{\partial \vec{F}}{\partial \vec{x}} = 0 \\
\\]
\\[
\frac{\partial \vec{F}}{\partial \vec{v}} = \beta I
\\]
Recalling that these objects are in fact $$3 \times 3$$ matricies
## Implementation

### LinearDragForce
```c++
class LinearDragForceGenerator
{
    private:
        GLfloat m_beta; // drag coefficent

    public:

        /**
            Constructs a LinearDragForceGenerator
        **/
        LinearDragForceGenerator();

        /**
            Constructs a LinearDragForceGenerator with the chosen drag coefficient
            @param beta drag coefficent
        **/
        LinearDragForceGenerator(GLfloat beta);

        /**
            @return the drag coefficent
        **/
        GLfloat GetDragCoeff();

        /**
            Sets the drag coefficent
            @param beta drag coefficient
        **/
        void SetDragCoeff(GLfloat beta);

        /**
            Adds a drag force whose magnitude is proportional to the drag coeffient
            @param entity Entity whom the force will be applied against
            @param F Force vector that will accumulate the constant force represented by this instance
        **/
        void AccumulateForce(std::shared_ptr<PhysicsEntity> entity, Vector3Gf &F) const;

        /**
            Adds the velocity jacobian
            @param entity Entity whom the force is acting on
            @param dF Matrix that will accumulate the derivative of force with respect to velocity
        **/
        void AccumulatedFdv(const std::shared_ptr<PhysicsEntity> entity, Eigen::Matrix<GLfloat,3,3> &dF) const;
};
```
```c++
LinearDragForceGenerator::LinearDragForceGenerator()
{
    m_beta = 1.0f;
}

LinearDragForceGenerator::LinearDragForceGenerator(GLfloat beta)
{
    m_beta = beta;
}

GLfloat LinearDragForceGenerator::GetDragCoeff()
{
    return m_beta;
}

void LinearDragForceGenerator::SetDragCoeff(GLfloat beta)
{
     m_beta = beta;
}

void LinearDragForceGenerator::AccumulateForce(std::shared_ptr<PhysicsEntity> entity, Vector3Gf &F) const
{
    GLfloat mass = entity->GetMass();
    Vector3Gf v = entity->GetVelocity();

    F += -m_beta*mass*v;
}

void LinearDragForceGenerator::AccumulatedFdv(const std::shared_ptr<PhysicsEntity> entity, Eigen::Matrix<GLfloat,3,3> &dF) const
{
    Eigen::Matrix<GLfloat,3,3> I = Eigen::Matrix<GLfloat,3,3>::Identity();

    dF += m_beta*I;
}
```

### NetForceAccumulator
```c++
class NetForceAccumulator
{
    private:
        // Same as before

        LinearDragForceGenerator m_drag_force;
        bool drag_on;

    public:
    // Same as before

    /**
        Enables/Disables velocity based drag
        @param enable true to turn on drag, false to turn off
    **/
    void EnableDrag(bool enable);

    /**
        Set the drag coefficent. Large values mean stronger drag
    **/
    void SetDragCoeff(GLfloat beta);

    // Same as before
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

void NetForceAccumulator::EnableDrag(bool enable)
{
    drag_on = enable;
}

void NetForceAccumulator::SetDragCoeff(GLfloat beta)
{
    m_drag_force.SetDragCoeff(beta);
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

    if(drag_on)
    {
        m_drag_force.AccumulateForce(entity_ptr,force);
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

    if(drag_on)
    {
        m_drag_force.AccumulatedFdv(entity_ptr,dFdv);
    }
}

```

## Results
For this simulation I turned off all forces except for the drag force. 
Then I gave the ball an initial velocity to the right.

<video controls>
    <source src="./images/Linear Drag/linear_drag.webm" type="video/webm"/>
</video>

As expected the ball starts out moving to the right and then slows to a stop.


Here's the [code](https://github.com/AdamSturge/Engine/tree/blog_linear_drag_force)
{% include links.html %}
