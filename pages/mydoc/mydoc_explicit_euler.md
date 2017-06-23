---
title: Explicit Euler
summary : "Solve Newton's equations of motiion"
sidebar: mydoc_sidebar
permalink: mydoc_explicit_euler.html
folder: mydoc
toc : true
---

## Overview
By the end of this article we will have our first legitimate physical simulation. 
Fundamentally the way our simulation will operate is my discretizing time into **time steps**, which are instants in time equally spaced apart. 
We will be using the symbol $$h$$ to represent how fine our temporal divisions are. Smaller $$h$$ values generally result in a more physically accurate simulation, but are more computationally demanding. 
Throughout the rest of this article we will be using a subscript $$i$$ to indicate the values at start of the time step, and a subscript $$f$$ to indicate values at the end of the time step.

Lets start out by reviewing some basic physics.

## Position
The rate of change in position $$\frac{\partial x}{\partial t}$$ is commonly refered to as velocity, which we denote $$v(t)$$. Writing out the full definition of the derivative we have

\\[
v(t) = \lim_{h \to 0}\frac{x(t+h) - x(t)}{h}
\\]


Denoting $$x_{i}$$ as $$x(t)$$, $$x_{f}$$ as $$x(t+h)$$, and $$v_{i}$$ as $$v(t)$$ we get

\\[
v_{i} = \lim_{h \to 0}\frac{x_{f} - x_{i}}{h}
\\]

Now comes the approximation. Instead of taking the true limit we will approximate the limit by taking $$h$$ to be a small but non-zero number.

\\[
v_{i} \approx \frac{x_{f} - x_{i}}{h}
\\]

Assuming we know the velocity at the start of the time step we can rearrange this exppression to solve for the unknown position $x_{f}$

\\[
x_{f} \approx x_{i} + hv_{i}
\\]

This wil be our update rule for position

## Velocity
To arrive at our velocity update rule we need to invoke [Newton's Second Law](https://en.wikipedia.org/wiki/Newton%27s_laws_of_motion)

{% include callout.html content="In an inertial reference frame $$F = \frac{\partial p}{\partial t}$$" type="primary"%}

Here $$F(t)$$ is the net [force](https://en.wikipedia.org/wiki/Force) acting on the object and $$p(t)$$ is the [momentum](https://en.wikipedia.org/wiki/Momentum), $$p=mv(t)$$. In general mass can also be a function of time, however in our case it is not. 
This reduces Newton's second law to the more familiar form 

\\[
F(t) = ma(t)
\\]

Where I've introduced the notation $$a(t) = \frac{\partial v}{\partial t}$$ for the rate of change in the velocity, or the **acceleration**

Applying the same trick we used for position we can approximate Newton's second law as

\\[
F_{i} \approx m\frac{v_{f}-v_{i}}{h}
\\]

As was the case for position assuming we know the net force acting on the object at the start of time step we can solve for the unknown velocity $$v_{f}$$

\\[
v_{f} \approx v_{i} + h\frac{F_{i}}{m}v_{i}
\\]

This is our update rule for velocity.

When put together these rules for position and velocity are referred to as the **Explicit Euler** method (for solving Newton's equations)

## Implementation
That's all well and good but we need to implement these update rules. To start we'll define an abstract superclass called TimeIntegrator. 
The name comes from the habit of people refering to these temporial update rules as "time intregating the simulation", although it has nothing to do with integration in the calculus sense.

You might be wondering why we are bothering with a superclass. Well here's a spoiler, there's more than one way to time integrate Newton's equations, and we'll be implmenting a few of them.

### Time Integrator

```c++
#include <Eigen/Core>
#include <GL/glew.h>
#include <vector3G.h>

#ifndef TIME_INTEGRATOR
#define TIME_INTEGRATOR
/**
    \brief An abstact base class for all grid based numerical ODE solvers for Newton's laws
**/
class TimeIntegrator
{
    protected:
        GLfloat m_dt; // Timestep size

    public:
         virtual ~TimeIntegrator(){};

        /**
            Public interface for all solvers. Computes position and velocity updates by solving Newton's equations of motion
            @param xi initial position of entity
            @param vi initial velocity of entity
            @param mass mass of entity 
            @param F force vector acting on the entity
            @param xf final position of entity. This will be updated with the new position for the next time step
            @param vf final velocity of entity. This will be updated with the new position for the next time step
        **/
         void StepForward(
            const Vector3Gf xi,
            const Vector3Gf vi,
            const GLfloat mass,
            const Vector3Gf F,
            Vector3Gf &xf,
            Vector3Gf &vf);

    private:
        /**
            Private implementation details for solver.
            @param xi initial position of entity
            @param vi initial velocity of entity
            @param mass mass of entity 
            @param F force vector acting on the entity
            @param xf final position of entity. This will be updated with the new position for the next time step
            @param vf final velocity of entity. This will be updated with the new position for the next time step
        **/
         virtual void Solve(
            const Vector3Gf xi,
            const Vector3Gf vi,
            const GLfloat mass,
            const Vector3Gf F,
            Vector3Gf &xf,
            Vector3Gf &vf) = 0;
};
#endif
``` 
Although we could have made do with a single public virtual method that would be violating the Single Responsibility Principle. See this [article]{https://www.spatial.com/blog/3d-software-development-kits/public-virtual-methods-bad-idea} for more details 

Here is the meager cpp file for TimeIntegrator

```c++
#include <time_integrator.h>
#include <assert.h>

void TimeIntegrator::StepForward(
            const Vector3Gf xi,
            const Vector3Gf vi,
            const GLfloat mass,
            const Vector3Gf F,
            Vector3Gf &xf,
            Vector3Gf &vf)

{
    assert(mass > 0.0f);

    Solve(xi,vi,mass,F,xf,vf);
}
```

Nothing exciting going on there.

### Explicit Euler
The magic happens when we provide a realization of TimeIntegrator

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

        /**
            Solves Newton's equations of motion
            @param xi intial position of entity
            @param vi initial velocity of entity
            @param mass mass of entity
            @param F force vector acting on the entity
            @param xf final poisition of entity. This will be updated with the new position for the next time step
            @param vf final velocity of entity. This will be ipdated with the new velocity for the next time step
        **/
        void Solve(
            const Vector3Gf xi,
            const Vector3Gf vi,
            const GLfloat mass,
            const Vector3Gf F,
             Vector3Gf &xf,
             Vector3Gf &vf);

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

void ExplicitEuler::Solve(
            const Vector3Gf xi,
            const Vector3Gf vi,
            const GLfloat mass,
            const Vector3Gf F,
            Vector3Gf &xf,
            Vector3Gf &vf)
{
    assert(mass != 0.0f);

    xf = xi + m_dt*vi;
    vf = vi + m_dt*(1/mass)*F;

};
```

## Results
Lets put Explicit Euler to work.
First lets give our sphere an intiial velocity of 0.3f to the right
```c++
Sphere sphere(1.0f, Vector3Gf(0.0f,0.0f,0.0f), Vector3Gf(0.3f,0.0f,0.0f),1.0f);
```

Next we'll declare an instance of ExplicitEuler with a time step size of 0.01
```c++
ExplicitEuler time_integrator(0.01);
```

Now we just need to replace the "fake" next position calculation in the render loop from [Lets get physical](mydoc_lets_get_physical) with

```c++
Vector3Gf xf,vf;
time_integrator.StepForward(
    sphere.GetPosition(),
    sphere.GetVelocity(),
    sphere.GetMass(),
    Vector3Gf(0.0f,0.0f,0.0f),
    xf,
    vf
);

sphere.SetNextPosition(xf);
sphere.SetNextVelocity(vf);
sphere.UpdateFromBuffers();
```

You should be greated with something like the following
<video controls>
    <source src="./images/Explicit Euler/Explicit_Euler.mp4" />
</video>

If so, congradulations! You've successfully completed your first real physics simulation.

If not, here's the full [code](https://github.com/AdamSturge/Engine/tree/blog_explicit_euler)

You may have noticed that we supplied a 0 vector for the net force. So what we've created is an experimental verification of Newton's first law

{% include callout.html content="An object at rest stays at rest and an object in motion stays in motion with the same speed and in the same direction unless acted upon by an unbalanced force." type="primary" %}
 

{% include links.html %}
