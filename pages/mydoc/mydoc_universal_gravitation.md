---
title: Universal gravitation
summary : "Implementing Newton's law of universal gravitation"
sidebar: mydoc_sidebar
permalink: mydoc_universal_gravitation.html
folder: mydoc
toc : true
---

## Overview
Newton's law of universal gravitation describes how 2 massive objects (that is objects with mass) are gravitationally attracted to one another.
Under our current understanding of physical law it is completely incorrect. 
However, remarkably, it is a highly accurate approximation for slow moving relatively light weight objects (here I use light weight in the cosmic sense). 
The true theory of gravity is Einstein's [general relativity](https://en.wikipedia.org/wiki/General_relativity), but even that has issues.
All that being said, we put a man on the moon with Newton's laws so it's certainly good enough for our purposes.

## Newton's Law of Universal Gravitation
Given two objects of masses $$m_{1}$$ and $$m_{2}$$ placed at positions $$\vec{x_{1}}$$ and $$\vec{x_{2}}$$ the gravitational force they experience is

\\[
\vec{F} = \frac{Gm_{1}m_{2}}{||\vec{n}||^{2}}\hat{n}
\\]

where $$G=6.673 \times 10^{-11}$$, $$\vec{n} = \vec{x_{2}}-\vec{x_{1}}$$, and $$\hat{n} = \frac{\vec{n}}{\|\vec{n}\|}$$. 
G is refered to the **Gravitational Constant** or sometimes Newton's constant. It is one of the fundemental numbers that defines our universe. 

There is some care required in the direction of $$\vec{n}$$. 
If we keep our labeling the same, then if object 1 experiences a force $$F$$ then object 2 experiences a force $$-F$$.
The negative sign has the effect of flipping the direction of the n vector so that object 1 moves towards object 2 and object 2 moves towards object 1.
Otherwise both objects would move in the same direction, which is obviously wrong for gravity.

Alternatively we can simply relable what is object 1 and what is object 2 when computing the gravitational force for object 2. 
This will have the net effect of switching the direction of $$\vec{n}$$ without affecting the magnitude of the force. 
We will utilize this symmetry in order to simplfy our code.

## Universal Gravitation Force Generator

```c++
#include <physics_entity.h>
#include <memory>
#include <vector3G.h>

#ifndef GRAVITY_FORCE_H
#define GRAVITY_FORCE_H

class GravityForceGenerator
{
    private :
        const GLfloat m_G = 6.673e-11;

    public :
        GravityForceGenerator();

        void AccumulateForce(const std::shared_ptr<PhysicsEntity> e1, const std::shared_ptr<PhysicsEntity> e2, Vector3Gf &F);
};

#endif
```

```c++
#include <gravity_force.h>

GravityForceGenerator::GravityForceGenerator(){};

void GravityForceGenerator::AccumulateForce(const std::shared_ptr<PhysicsEntity> e1, const std::shared_ptr<PhysicsEntity> e2, Vector3Gf &F)
{
    GLfloat m1 = e1->GetMass();
    GLfloat m2 = e2->GetMass();

    Vector3Gf x1 = e1->GetPosition();
    Vector3Gf x2 = e2->GetPosition();

    Vector3Gf n = x2 - x1;
    GLfloat r = n.norm();
    n = n/r;

    if(r >= 0.001f)
    {
        F += (m_G*m1*m2/(r*r))*n;
    }

};

```
You'll notice that if the objects are too close to each other we don't compute the gravitational force. 
That is simply to avoid numberical instabilities when [dividing by small numbers](https://books.google.ca/books?id=5XappvcENCMC&pg=PA10&lpg=PA10&dq=dividing+small+numbers+numerically+unstable&source=bl&ots=RUHjXM2puA&sig=gtiG2rWbglDwxkvWicS8XpaPzrU&hl=en&sa=X&ved=0ahUKEwjpx_q28dnUAhXp5oMKHauxAgUQ6AEINjAD#v=onepage&q=dividing%20small%20numbers%20numerically%20unstable&f=false).
Also you don't want to divide by 0 of course.

## Net force
After adding a new member variable to [Scene](https://github.com/AdamSturge/Engine/blob/blog_universal_gravitation/include/scene.h) we can update our net force computation.

```c++
void Scene::ComputeNetForce(const std::shared_ptr<PhysicsEntity> entity_ptr, Vector3Gf &force)
{
    m_constant_force_generator.AccumulateForce(entity_ptr,force);
    
    for(std::shared_ptr<PhysicsEntity> other_entity_ptr : m_physics_entity_ptrs)
    {       
        if(other_entity_ptr != entity_ptr)
        {
            m_gravity_force_generator.AccumulateForce(entity_ptr,other_entity_ptr,force);
        }
    }
};
```
Where we've been careful not to allow gravitational self interaction.

## Orbits
Now that we've implemented gravitational attraction lets do something fun. 
Given a body of mass $$M$$ the velocity a second body needs to orbit it is

\\[
    v = \sqrt{\frac{GM}{||\vec{x_{2}} - \vec{x_{1}}||}}
\\]

I won't go into a derivation of this formula as it requires a little knowledge about [centripetal force](https://en.wikipedia.org/wiki/Centripetal_force), which we haven't talked about.
So just accept it as fact for now.

Using this formula we can set up our scene so one sphere orbits the other.

```c++
    std::shared_ptr<Sphere> sphere1_ptr(new Sphere(1.0f, Vector3Gf(0.0f,0.0f,0.0f), Vector3Gf(0.0f,0.0f,0.0f), 1.0f)); 
    scene.AddPhysicsEntity(sphere1_ptr); 
    scene.AddModel(sphere1_ptr); 
 
    std::shared_ptr<Sphere> sphere2_ptr(new Sphere(2.0f, Vector3Gf(5.0f,0.0f,0.0f), Vector3Gf(0.0f,0.0f,0.0f), 1e12)); 
    scene.AddPhysicsEntity(sphere2_ptr); 
    scene.AddModel(sphere2_ptr); 
 
    Vector3Gf orbital_velocity( 
        0.0f, 
        sqrt((6.673e-11)*sphere2_ptr->GetMass()/((sphere2_ptr->GetPosition() - sphere1_ptr->GetPosition()).norm())), 
        0.0f 
    ); 
 
    sphere1_ptr->SetNextVelocity(orbital_velocity); 
    sphere1_ptr->UpdateFromBuffers(); 
```

If everything works correctly you should see this

<video controls>
    <source src="./images/Universal Gravitation/orbit.mp4"  type="video/mp4"/>
</video>

Whoops! The orbit isn't stable. It keeps growing with each pass. Why is this happening? Could there a be a problem with the code. Or worse yet physics?! 
What's actually happening is more subtle than either of those possibilities. This is a limitation of Explicit Euler. It does not conserve energy.
Over time it will either add so much energy into your system that it blows up, or bleed all the energy away until the scene becomes static. So what now? Should we pack up our bags and go home?
No, remember when I said we'd be implementing other time integrators back when we introduced Explict Euler? Well this is why. So over the next few articles we are going to do a deep dive on some numerical differential equation solvers that will be useful.

As always, the [code](https://github.com/AdamSturge/Engine/tree/blog_universal_gravitation) is here if you want to look at it

{% include links.html %}
