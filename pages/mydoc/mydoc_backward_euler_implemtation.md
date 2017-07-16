---
title: Backawrd Euler (implementation)
summary : "Implementation details for Backward Euler"
sidebar: mydoc_sidebar
permalink: mydoc_backward_euler_implementation.html
folder: mydoc
toc : true
---

## Overview
We're finally here, in this article we will implement [Backward Euler](mydoc_backward_euler_implementation.html). 
Recall the equation we are trying to solve is
\\[
y_{n+1} - y_{n} - hf(t_{n+1},y_{n+1}) = 0
\\]
The way we intend to solve this implcit equation is through [Newton's Method](mydoc_newtons_method.html). 
In order to use Newton's Method we need to compute **force jacobians**

## Force Jacobians
Force jacobians in 3 dimensions are $$3 \times 3$$ matrices
\\[
\frac{\partial \vec{F}}{\partial \vec{x}\_{n+1}}
=
\begin{bmatrix}
\frac{\partial F_{x}}{\partial x_{n+1,x}} & \frac{\partial F_{x}}{\partial x_{n+1,y}} & \frac{\partial F_{x}}{\partial x_{n+1,z}} \\\
\frac{\partial F_{y}}{\partial x_{n+1,x}} & \frac{\partial F_{y}}{\partial x_{n+1,y}} & \frac{\partial F_{y}}{\partial x_{n+1,z}} \\\
\frac{\partial F_{z}}{\partial x_{n+1,x}} & \frac{\partial F_{z}}{\partial x_{n+1,y}} & \frac{\partial F_{z}}{\partial x_{n+1,z}}
\end{bmatrix}
\\]

\\[
\frac{\partial \vec{F}}{\partial \vec{v}\_{n+1}}
=
\begin{bmatrix}
\frac{\partial F_{x}}{\partial v_{n+1,x}} & \frac{\partial F_{x}}{\partial v_{n+1,y}} & \frac{\partial F_{x}}{\partial v_{n+1,z}} \\\
\frac{\partial F_{y}}{\partial v_{n+1,x}} & \frac{\partial F_{y}}{\partial v_{n+1,y}} & \frac{\partial F_{y}}{\partial v_{n+1,z}} \\\  
\frac{\partial F_{z}}{\partial v_{n+1,x}} & \frac{\partial F_{z}}{\partial v_{n+1,y}} & \frac{\partial F_{z}}{\partial v_{n+1,z}}
\end{bmatrix}
\\]

We will compute the net force jacobian in the same manner as wel compute the net force, accumulation. 
To that end we will need to know the force jacobians of all the forces in our simulation.

### Constant Force
The constant force is $$\vec{F} = m\vec{a}$$ for a constant vector $$\vec{a}$$.
This force has no explicit dependance on $$\vec{x}$$ or $$\vec{v}$$ so the force jacobians are both the zero matrix.

### Gravitational Force
The gravitational force is a function of position $$\vec{x}$$ but not velocity $$\vec{v}$$. 
So $$\frac{\partial \vec{F}}{\partial \vec{v}_{n+1}}$$ is the zero  matrix and
\\[
\frac{\partial \vec{F}}{\partial \vec{x}\_{n+1}} = -\frac{Gm_{1}m_{2}}{\|\|\vec{x}\_{2} - \vec{x}\_{1}\|\|^{3}}(I - 3\hat{n}\hat{n}^{T})
\\]
where $$\hat{n}= \frac{\vec{x}_{2} - \vec{x}_{1}}{|| \vec{x}_{2} - \vec{x}_{1} ||} \in \mathbb{R}^{3}$$

## Implementation

We will be adding two new methods to our force generators (where appropriate), *AccumulatedFdx* and *AccumulatdFdv*. 
So far neither of our 2 forces depend on velocity so we won't be implementing the latter method just yet. 
*ConstantForceGenerator* has no contribution to  make to the net force jacobian so it will remain the same. 
However *GravityForceGenerator* now contributes to the position jacobian like so

### Gravity Force Jacobian
```c++
void GravityForceGenerator::AccumulatedFdx(const std::shared_ptr<PhysicsEntity> e1, const std::shared_ptr<PhysicsEntity> e2, Eigen::Matrix<GLfloat,3,3> &dF) const
{
    GLfloat m1 = e1->GetMass();
    GLfloat m2 = e2->GetMass();

    Vector3Gf x1 = e1->GetPosition();
    Vector3Gf x2 = e2->GetPosition();

    Vector3Gf n = x2 - x1;
    GLfloat r = n.norm();
    n = n/r;

    Eigen::Matrix<GLfloat, 3, 3> N = n*n.transpose();
    Eigen::Matrix<GLfloat,3,3> I = Eigen::Matrix<GLfloat,3,3>::Identity();

    dF += -(m_G*m1*m2/std::pow(r,3))*(I - 3*N);

}

```

### Scene
In scene we will be adding a new method to compute the net jacobian.
```c++ 
void Scene::ComputeForceJacobian(const std::shared_ptr<PhysicsEntity> entity_ptr, Eigen::Matrix<GLfloat,3,3> &dFdx, Eigen::Matrix<GLfloat,3,3> &dFdv) const
{
    for(std::shared_ptr<PhysicsEntity> other_entity_ptr : m_physics_entity_ptrs)
    {
        if(other_entity_ptr != entity_ptr)
        {
            m_gravity_force_generator.AccumulatedFdx(entity_ptr,other_entity_ptr,dFdx);
        }
    }
};
```

### Backward Euler
This should look very familar at this point
```c++
#ifndef BACKWARD_EULER_H
#define BACKWARD_EULER_H
#include "time_integrator.h"

/**
    \brief Implementation of Backward Euler method for solving Newton's Laws
**/
class BackwardEuler : public TimeIntegrator
{
    public:
        BackwardEuler();

        BackwardEuler(GLfloat dt);

    private:
        /**
            Private implementation details for solver.
            @param scene scene containing entities to be integrated forward
            @param entity_ptr pointer to the entity being updated
        **/
        void Solve(const Scene& scene,const std::shared_ptr<PhysicsEntity> entity_ptr);

        /**
            Computes the Jacobian of the update rule for use in Newton's Method
            @param mass mass of entity being simulated
            @param dFdx position force jacobian
            @param dFdx velocity force jacobian
            @param J matrix that will be modified to contain the update rule jacobian
        **/
        void ComputeJacobian(
                         const GLfloat mass,
                         const Eigen::Matrix<GLfloat,3,3> dFdx,
                         const Eigen::Matrix<GLfloat,3,3> dFdv,
                         Eigen::Matrix<GLfloat,6,6> &J) const;
};
#endif

```

Recalling the form of the jacobian $$\nabla G$$ implementing *ComputeJacobian* is fairly straightforward
\\[
\nabla G =
\begin{bmatrix}
I & hI \\\
-\frac{h}{m}\frac{\partial F}{\partial x_{n+1}} & I - \frac{h}{m}\frac{\partial F}{\partial v_{n+1}}
\end{bmatrix}
\\] 

```c++
void BackwardEuler::ComputeJacobian(
                                    const GLfloat mass,
                                    const Eigen::Matrix<GLfloat,3,3> dFdx,
                                    const Eigen::Matrix<GLfloat,3,3> dFdv, 
                                    Eigen::Matrix<GLfloat,6,6> &J) const
{
    Eigen::Matrix<GLfloat,3,3> I = Eigen::Matrix<GLfloat,3,3>::Identity();
    J.block(0,0,3,3) = I;

    J.block(0,3,3,3) = m_dt*I;

    J.block(3,0,3,3) = -(m_dt/mass)*dFdx;

    J.block(3,3,3,3) = I - (m_dt/mass)*dFdv;
};

```

The implementation of *Solve* is our most complicated yet
```c++
void BackwardEuler::Solve(const Scene& scene,const std::shared_ptr<PhysicsEntity> entity_ptr)
{
    Vector3Gf xi = entity_ptr->GetPosition();
    Vector3Gf vi = entity_ptr->GetVelocity();
    GLfloat mass = entity_ptr->GetMass();

    // Set initial guess z_0 by using Symplectic Euler
    Vector3Gf F;
    F.setZero();
    scene.ComputeNetForce(entity_ptr,F);
    Vector3Gf vf = vi + m_dt*F/mass;
    Vector3Gf xf = xi + m_dt*vf;

    entity_ptr->SetPosition(xf);
    entity_ptr->SetVelocity(vf);

    GLfloat diff = 100.0f;
    Eigen::Matrix<GLfloat,6,6> J;
    Eigen::Matrix<GLfloat,3,3> dFdx,dFdv;
    Eigen::Matrix<GLfloat,6,1> G;    
    Eigen::Matrix<GLfloat,6,1> delta;
    for(int i = 0; i < 3 && diff > 1e-9; ++i)
    {   
        // Compute Jacobian at current guess
        J.setZero();
        dFdx.setZero();
        dFdv.setZero();
        scene.ComputeForceJacobian(entity_ptr,dFdx,dFdv);
        ComputeJacobian(mass,dFdx,dFdv,J);

        // Compute force at current guess
        F.setZero();
        scene.ComputeNetForce(entity_ptr,F);

        // Compute G for current guess
        G.topRows(3) = xf - xi - m_dt*vf;
        G.bottomRows(3) = vf - vi - m_dt*F/mass;

        // Solve for delta
        delta = J.ldlt().solve(-G);

        // Update guess
        xf = xf + delta.topRows(3);
        vf = vf + delta.bottomRows(3);

        // Load guess into entity for Jacobian and force computations
        entity_ptr->SetPosition(xf);
        entity_ptr->SetVelocity(vf);

        diff = delta.squaredNorm();
     }

    entity_ptr->SetPosition(xi);
    entity_ptr->SetVelocity(vi);

    entity_ptr->SetNextPosition(xf);
    entity_ptr->SetNextVelocity(vf);
```
If you've been following along you may have noticed that a slipped a small *PhysicsEntity* update into here.
Backward Euler was the breaking point for me so I finally implemented *SetPosition* and *SetVelocity*. 
Initially I had planned to avoid implementing these to prevent accidently calling the wrong function when setting the position and velocity for the next time step. 
But it's become apparent that is too limiting when dealing with more complicated time integrators than Explicit Euler.

Of note here is that we are setting our initial guess using Symplectic Euler.
A good initial guess can dramtically reduce the runtime of Newton's Method so it's generally worth it to spend a little computation time here.
Also our stopping criteria is when the updates to our guess become sufficently small, or we've performed $$3$$ iterations, whichever comes first. 

## Results
Here's the results of all our work

<video controls>
    <source src="./images/Backward Euler implementation/orbit_decay.webm" type="video/webm" />
</video>

Egads! Turns out Backward Euler suffers from the same energy drift problem as Explicit Euler, except where one explodes the other collapses. 
This is because Backward Euler is not a symplectic method.
However there is a way to build a symplectic integrator out of Backward Euler. 
This would be the **Midpoint Method**, and it will be our last integrator.

[code](https://github.com/AdamSturge/Engine/tree/blog_backward_euler)

{% include links.html %}
