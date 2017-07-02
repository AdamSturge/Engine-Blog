---
title: Verlet Integrator
summary : "A more accurate symplectic integrator"
sidebar: mydoc_sidebar
permalink: mydoc_verlet.html
folder: mydoc
toc : true
---

## Overview
In this article we will be learning how the accuracy of numerical methods is discussed and how this applies to a new integrator, Verlet Integration. 

## Local error
The local error of a numerical method is the error incurred from one step of the integration to the next, ie between $$y_{n}$$ and $$y_{n+1}$$.
When performing this kind of error analysis we assume that the previous step, $$y_{n}$$, is the exact answer, even if it isn't really.  
First let me introduce the following notation: $$y(t_{n})$$ will refer to the exact answer at time $$t_{n}$$ and $$y_{n}$$ will refer to the approximate answer that our numerical method outputs. 

We start out with the [Taylor series](https://en.wikipedia.org/wiki/Taylor_series) expansion of $$y(t_{n}+h)$$ about $$t_{n}$$
\\[
    y(t_{n}+h) = y(t_{n}) + \dot{y}(t_{n})h + \frac{1}{2}\ddot{y}(t_{n})h^{2} + \frac{1}{6}\\dddot{y}(t_{n})h^{3} + \cdots
\\]

Where I have adopted the notation common in physics literature that $$\dot{f}(t) = \frac{\partial f}{\partial t}$$. 
We can more compactly write this relation as

\\[
    y(t_{n}+h) = y(t_{n}) + \dot{y}(t_{n})h + \frac{1}{2}\ddot{y}(t_{n})h^{2} + \mathcal{O}(h^{3})
\\]

where I've made use of the "Big O" notation to show that the next term would be of order $$h^{3}$$. 
Technically we've lost information writing it in this way since $$\mathcal{O}(h^{3})$$ doesn't tell us anything about the coefficent in front of $$h^{3}$$, but it highlights how people talk about these methods. 

If we were to write
\\[
y_{n+1} = y_{n} + \dot{y}(t_{n})h + \frac{1}{2}\ddot{y}(t_{n})h^{2} 
\\]
then
\\[
y(t_{n}+h) - y_{n+1} = \mathcal{O}(h^{3})
\\]
and we would say that the local error in our estimate is $$\mathcal{O}(h^{3})$$.
Genreally the local error rate isn't really reported on its own and is more seen as a means to get an estimate of the global error rate, which we will talk about next.

## Global error
The global error is the total error accrued between the start of the simulation and an arbitrary simulation step $$t_{n}$$, assuming we have an exact answer for $$y(t_{0})$$.
Meaning it is the sum of all the local errors made between the first time step and the time step in question. 
In general it is much harder to find an exact relation or even order for this error. 
Often a bound on the order of the global error is the best that can be done.
While technically it would be more accurate to say "the global error is no worse than $$\mathcal{O}(h^{2})$$" people tend to say "the global error is $$\mathcal{O}(h^{2})$$".
This [pdf](http://www.math.unl.edu/~gledder1/Math447/EulerError) has a detailed walkthrough of local and global error analysis for Explict Euler

## Verlet Integration {#verlet_integration}
Verlet integration includes terms up to second order for $$x(t)$$

\\[
x_{n+1} = x_{n} + v_{n}h + \frac{1}{2}a_{n}h^{2} = x_{n} + v_{n}h + \frac{1}{2}\frac{F(x_{n})}{m}h^{2}
\\] 

Nothing too exciting going on there. 

The velocity update rule is more interesting.
First we are going to do absolutely nothing in a useful way. 
Lets write
\\[
v(t+h) = v(t) + \int\limits_{t}^{t+h}a(\tau)d\tau
\\]
Recalling that $$a(t) = \dot{v}(t)$$ this equation simplifies to $$v(t+h) = v(t) + (v(t+h)-v(t))$$.
Now we are going to use the [trapazoid rule](https://en.wikipedia.org/wiki/Trapezoidal_rule) to approximate the integral on the right.
This gives us the update rule
\\[
v_{n+1} = v_{n} + \frac{1}{2}(a_{n} + a_{n+1})h = v_{n} + \frac{1}{2m}(F(x_{n}) + F(x_{n+1}))h 
\\]

As you can see the velocity update depends on the force evaluated at the next time step in a similar manner to Symplectic Euler (except in that case it was position as a function of the next velocity).

### Comparision to Symplectic Euler
The Verlet update rules I've given you (sometimes refered to as velocity Verlet) have the following error orders:

**Verlet**

|Quanity|Local error|Global error|
|-------|--------|---------|
|$$x(t)$$|$$\mathcal{O}(h^{4})^{*}$$|$$\mathcal{O}(h^{2})$$|
|$$v(t)$$|$$\mathcal{O}(h^{2})$$|$$\mathcal{O}(h^{2})$$|

<sub>
\* You might think it'd be $$\mathcal{O}(h^{3})$$ but it turns out the $$h^{3}$$ terms [cancel out](https://en.wikipedia.org/wiki/Verlet_integration#Discretization_error). 
Just use $$v_{n} = \frac{x_{n}-x_{n-1}}{h}$$ in the position equation
</sub>

Whereas Symplectic Euler has the following error orders:

**Symplectic Euler**

|Quanity|Local error|Global error|
|-------|--------|---------|
|$$x(t)$$|$$\mathcal{O}(h^{2})$$|$$\mathcal{O}(h)$$|
|$$v(t)$$|$$\mathcal{O}(h^{2})$$|$$\mathcal{O}(h)$$|

As you can see Verlet has a better global error rate than Symplectic Euler. 
What's more Verlet Integration is a symplectic method so we don't lose this important property by switching to Verlet.
The cost is naturally that it takes more function calls and arthmatic operations to perform Verlet than it does with Symplectic Euler.


## Implementation
I'll only give the *Solve* method details as the rest is the same as previous integrators.
```c++
void Verlet::Solve(const Scene& scene,const std::shared_ptr<PhysicsEntity> entity_ptr)
{
    const Vector3Gf xi = entity_ptr->GetPosition();
    const Vector3Gf vi = entity_ptr->GetVelocity();
    const GLfloat mass = entity_ptr->GetMass();

    Vector3Gf Fi,Ff,ai,xf,af,vf;

    Fi.setZero();
    scene.ComputeNetForce(entity_ptr,Fi);

    ai = (1/mass)*Fi;
    xf = xi + m_dt*vi + 0.5f*m_dt*m_dt*ai;

    entity_ptr->SetNextPosition(xf);
    entity_ptr->UpdateFromBuffers(); // Load xf into position slot for computing F(x(t+h))

    Ff.setZero();
    scene.ComputeNetForce(entity_ptr,Ff); // Compute F(x(t+h))

    entity_ptr->SetNextPosition(xi); // Load xi back into position slot as to not affect force calulations for other entities
    entity_ptr->UpdateFromBuffers();

    af = (1/mass)*Ff;
    vf = vi + 0.5f*m_dt*(ai + af);

    entity_ptr->SetNextPosition(xf);
    entity_ptr->SetNextVelocity(vf);
};                                            
```
Everything proceeds as detailed above. 
The only somewhat annoying part is the fact that we need to load $$x_{n+1}$$ from buffer to compute $$F(x_{n+1})$$ then load $$x_{n}$$ back so as to not mess up the updates of other entities in the scene.
This is in fact the reason we redesined TimeIntegrator back in [Updated TimeIntegrator](mydoc_updated_time_integrator.html).
We needed the ability to compute accelerations based on partially updated quantities like $$(x_{n+1},v_{n})$$.
If we were designing an engine for a real game we'd make a choice as to what integrator we want to use from the start. 
Then design our architecture to maximize the efficency of these updates as they will be called often. 
In our case however we are taking a more plug and play style approach to time integrators so naturally we are going to pay a price in efficiency.

## Results
In order to test out our new method we need to crank up the forces involved in our toy orbit problem.
We can accomplish this by increasing the mass of our central planet to $$10^{17}$$. 
Higher forces mean that the position and velocity change a lot between time steps, giving our more accurate method a chance to shine. 
Since the error is a function of $$h$$ we'll keep it fairly large (in comparision to the forces involved) at $$h = 0.0095$$.

### Symplectic Euler
<video controls>
    <source src="./images/Verlet/symplectic euler.mp4" />
</video>
As you can see the errors build up until the orbiting body is flung off into space.

### Verlet
<video controls>
    <source src="./images/Verlet/verlet.mp4" />
</video>
Although the moon moves a great distance with each simulation step, the $$\mathcal{O}(h^{2})$$ global error keeps the orbit from becoming unstable during the simulation.

Here's the [code](https://github.com/AdamSturge/Engine/tree/blog_verlet)

{% include links.html %}
