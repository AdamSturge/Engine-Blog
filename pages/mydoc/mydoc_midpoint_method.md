---
title: Implicit Midpoint Method
summary : "A symplectic A-stable method"
sidebar: mydoc_sidebar
permalink: mydoc_midpoint_method.html
folder: mydoc
toc : true
---

## Overview
After all our work on [Backward Euler](mydoc_backward_euler_implementation.html) it turns out to bleed away energy in our orbiti simulation.
Fortunately there's a easy to implement symplectic method that uses Backward Euler has a subroutine. 
The so called [Implicit Mipoint Method](https://en.wikipedia.org/wiki/Midpoint_method). 
The Implicit Midpoint Method is the lowest tier of [Gauss-Legendre Methods](https://en.wikipedia.org/wiki/Gauss%E2%80%93Legendre_method).
All the Guass-Legendre methods are symplectic and A-stable. 
This makes them very well behavied integrators. 

The Implicit Midpoint rule is
\\[
y_{n+1} = y_{n} + hf(t_{n} + \frac{h}{2}, \frac{1}{2}(y_{n} + y_{n+1}))
\\]

If you are like me the notation $$f(t_{n} + \frac{h}{2}, \frac{1}{2}(y_{n} + y_{n+1}))$$ confuses the hell out of you. 
What does it mean when the $$t$$ parameter is evaluted at a different point than the $$y$$ parameter if $$y$$ depends on $$t$$?
If you ignore the implicit dependence of $$y$$ on $$t$$ then you get the [implciit trapazoid method](https://en.wikipedia.org/wiki/Trapezoidal_rule_(differential_equations)), which to my knowledge is not symplectic.

Turns out one way out of this quagmire is through variable substitution. 

Let $$y_{n+\frac{1}{2}} = \frac{1}{2}(y_{n} + y_{n+1})$$.
We can write this half-point "exactly" because the midpoint method is a first order method.
As such it assumes straight line motion between time steps.

Plugging these equations into our update rule yields
\\[
2 y_{n+\frac{1}{2}} - y_{n} = y_{n} + hf(t_{n} + \frac{h}{2},y_{n+\frac{1}{2}} ) \iff
 y_{n+\frac{1}{2}} = y_{n} + \frac{h}{2}f(t_{n} + \frac{h}{2},y_{n+\frac{1}{2}} ) 
\\]

This is Backward Euler with a step size of $$\frac{h}{2}$$.

Once we have $$y_{n+\frac{1}{2}}$$ we can solve for $$y_{n+1} = 2y_{n+\frac{1}{2}} - y_{n}$$.

### Implementation
The implementation details for this method are sparse.
Only thing worth calling out is that we will be needing getters for the next position and next velocity buffers, whose implementation I leave to you.
```c++
fndef MIDPOINT_METHOD_H
#define MIDPOINT_METHOD_H
#include "time_integrator.h"
#include "backward_euler.h"
/**
    \brief Implementation of Midpoint method for solving Newton's Laws
**/
class MidpointMethod : public TimeIntegrator
{
    public:
        MidpointMethod();

        MidpointMethod(GLfloat dt);

    private:
        BackwardEuler m_backward_euler;

        /**
            Private implementation details for solver.
            @param scene scene containing entities to be integrated forward
            @param entity_ptr pointer to the entity being updated
        **/
        void Solve(const Scene& scene,const std::shared_ptr<PhysicsEntity> entity_ptr);
       
};
#endif
```

```c++
#include "midpoint_method.h"

MidpointMethod::MidpointMethod()
{
    m_dt = 0.1;
    m_backward_euler = BackwardEuler(0.5f*m_dt);
}

MidpointMethod::MidpointMethod(GLfloat dt)
{
    m_dt = dt;
    m_backward_euler = BackwardEuler(0.5f*m_dt);
}

void MidpointMethod::Solve(const Scene& scene,const std::shared_ptr<PhysicsEntity> entity_ptr)
{
    Vector3Gf xi = entity_ptr->GetPosition();
    Vector3Gf vi = entity_ptr->GetVelocity();

    m_backward_euler.StepForward(scene,entity_ptr);

    Vector3Gf xhalf = entity_ptr->GetNextPosition();
    Vector3Gf vhalf = entity_ptr->GetNextVelocity();

    Vector3Gf xf = 2*xhalf - xi;
    Vector3Gf vf = 2*vhalf - vi;

    entity_ptr->SetNextPosition(xf);
    entity_ptr->SetNextVelocity(vf);

}
```

## Results

<video controls>
    <source src="./images/Implicit Midpoint Method/midpoint_method.webm" type="video/webm"/>
</video>

It seems crazy that this works.
Afterall it just looks like we are doing Backward Euler with half the step size. 
However that is not the case, as you can see below

<video controls>
    <source src="./images/Implicit Midpoint Method/backward_euler_half_step.webm" type="video/webm"/>
</video>

So what's going on here?
Well one way to think of it is that Backward Euler alone tends to damp our system and bleed away energy.
On the other hand Explicit Euler tends to exaggerate our system and ramp up the energy. 
So what if we combined these two in such a way that one more or less canceled out the other. 
That's what implicit midpoint is doing. 
The first half of the simulation is performed using Backward Euler. 
The second half, solving for $$y_{n+1}$$ from $$y_{n+\frac{1}{2}}$$ and $$y_{n}$$ is equivalent to using Explicit Euler starting at $$y_{n+\frac{1}{2}}$$.
Both together are symplectic, where as alone neither is. Neat!

[code](https://github.com/AdamSturge/Engine/tree/blog_midpoint_method)

{% include links.html %}
