---
title: Rotational Explicit Euler
summary : "A simple discrete update for rotational dynamics"
sidebar: mydoc_sidebar
permalink: mydoc_rotational_explicit_euler.html
folder: mydoc
toc : true
---

## Overview
In the last article we talked about the dynamics of rotating bodies. 
At the end we came up with a differential equation but we didn't discuss how to solve it on a computer.
That will be the topic of today's discussion. 

## Explicit Euler
First, a reminder of the equation we are trying to solve
\\[
\frac{\partial}{\partial t}\tilde{q} = \frac{\tilde{\omega}}{2}\tilde{q} 
\\]
It should come as no surprise to those of you who've read the articles on linear motion that the simpliest method is [Explicit Euler](./mydoc_explicit_euler.html). 
And honestly, as far as I can tell, nobody has bothered to ever talk about using any other method for this kind of work. 
Which is interesting in and of itself. 
My guess is that the more advanced methods become tricker to deal with since quaternions aren't commutative, and people just don't want to deal with that if regular old Explicit Euler seems good enough. 
Anyways Explicit Euler involves writing the left hand side as
\\[
\lim_{h \to 0 }\frac{\tilde{q}(t+h) - \tilde{q}(t)}{h} = \frac{\tilde{\omega}}{2}\tilde{q}(t) 
\\]
We can rearrage this into
\\[
\lim_{h \to 0}\tilde{q}(t+h) = \tilde{q}(t) + \lim_{h \to 0}\frac{h}{2}\tilde{\omega}\tilde{q}(t)
\\]
Finally we approximate the limits by setting our timestep $$h$$ to be small
\\[
\tilde{q}\_{f} =  \tilde{q}\_{o} + \frac{h}{2}\tilde{\omega}\tilde{q}\_{o}
\\]
Where $$\tilde{q}_{o}$$ is the orientation for the previous timestep and $$\tilde{q}_{f}$$ is our new orientation

## Implementation
First we need to add an angular velocity component to our PhysicsEntity.
This is mearly a simple 3D vector in the same vein as linear velocity.
I won't bother putting that here, as it's trival. 
For now lets just insert the update rule at the bottom of our existing Explict Euler solver
```c++
void ExplicitEuler::Solve(
            const NetForceAccumulator& net_force_accumulator,
        const std::vector<std::shared_ptr<PhysicsEntity>> &entity_ptrs,
            const std::shared_ptr<PhysicsEntity> entity_ptr)
{
    const Vector3Gf xi = entity_ptr->GetPosition();
    const Vector3Gf vi = entity_ptr->GetVelocity();
    const GLfloat mass = entity_ptr->GetMass();
    const Quaternion oi = entity_ptr->GetOrientation();
    const Vector3Gf  wi = entity_ptr->GetAngularVelocity();

    Vector3Gf F;
    F.setZero();
    net_force_accumulator.ComputeNetForce(entity_ptrs,entity_ptr,F);

    Vector3Gf xf = xi + m_dt*vi;
    Vector3Gf vf = vi + m_dt*(1/mass)*F;

    entity_ptr->SetNextPosition(xf);
    entity_ptr->SetNextVelocity(vf);

    Quaternion of = oi + 0.5*m_dt*Quaternion(wi)*oi;

    entity_ptr->SetNextOrientation(of);

};
```

## Results
<video controls>
  <source src="./images/Rotational Explicit Euler/rotational_explicit_euler.webm" type="video/webm"/>
</video>
Now our cube spins.

As a side note I'm begining to feel like the normals for the cube are messed up based on how it's being lit up.
I should implement a model loading library soon

{% include links.html %}
