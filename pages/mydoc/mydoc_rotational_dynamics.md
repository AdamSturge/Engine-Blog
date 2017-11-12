---
title: Rotational Dynamics
summary : "Introduction to computing rotations as a function of forces"
sidebar: mydoc_sidebar
permalink: mydoc_rotational_dynamics.html
folder: mydoc
toc : true
---

## Overview
Now that we can represent an orientation with a quaternion the next step is to update that quaternion over time. 
To do that we will need a governing equation.
For linear motion we used Newton's second law, so it's not unreasonable to expect something analagous to apply here. 

## Orientation vs rotation
Before we move on I want to clarify some terminology.
An **orientation** refers to how an object is oriented about a given axis.
A **rotation** is the act of updating your orientation.
Put another way, an orientation is a state, whereas a rotation is a transformation.
This is why rotation matricies are a common way to represent rotations, a transformation that acts on a vector is most commonly represented as a matrix afterall.
It's a subtle distinction and I'm as guilty as anyone of conflating these ideas from time to time.
So to be clear, the quaternion we've stored in our entity so far represents **the orientation of the entity**.
In this and the following few articles we will be discussing how to update our orientation, ie how to physically compute a rotation. 

## Eulers second law
In order to compute our rotation we don't need to work with quaternions until the very end.
You might be thinking, "hey what about all that gimbal lock stuff!". 
The physics of rotation is fundementally $$3$$D, so $$\mathbb{R}^{3}$$ is fine. 
Gimble lock is a byproduct of a poor method of storing and manipulating rotations (Euler Angles).
This [article](https://mathoverflow.net/questions/95902/the-gimbal-lock-shows-up-in-my-quaternions) has an interesting discussion of gimbal lock from a topological point of view.

Without further ado here's our formula, [Euler's second law](https://en.wikipedia.org/wiki/Euler%27s_laws_of_motion#Euler.27s_second_law)
\\[
    \vec{\tau} = I\dot{\vec{\omega}}
\\]
$$\vec{\tau} \in \mathbb{R}^{3}$$ refers to a qauntity called [torque](https://en.wikipedia.org/wiki/Torque), which is the rotatiional version of force. 
$$I \in \mathbb{R}^{3 \times 3}$$ refers to the [inertial tensor](https://en.wikipedia.org/wiki/Moment_of_inertia) (also sometimes called the "moment of inertia"). This is the rotational version of mass and is represented by a matrix.
Lastly $$\vec{\omega} \in \mathbb{R}^{3}$$ is how we are going to represent our rotation. It refers to the quantity called [angular velocity](https://en.wikipedia.org/wiki/Angular_frequency).
By way of reminder a dot above a vector indicates a time derivative. Making $$\dot{\vec{\omega}}$$ our rotational analog of linear acceleration.

It's the subject of future articles to develop these ideas further, so for now lets put them aside and press forward.

## Acting an angular velocity on an orientation.
The question now naturally arises, assuming we have some means to compute $$\vec{\omega}$$ how do we apply it to our orientation, which is a quaternion?
Without loss of generality assume our orientation quaternion $$\tilde{q}$$ is normalized.
All normalized quaternions correspond to a rotation of some angle $$\theta$$ about some axis $$\vec{u} = (u_{x},u_{y},u_{z})$$.
With these quantities in hand the quaternion can be written 
\\[
\tilde{q} = e^{\frac{\theta}{2}\vec{u}}
\\]
by an extension of [Euler's formula](https://en.wikipedia.org/wiki/Euler%27s_formula) to the quaternions. 
This is by no means obvious and comes from the fact that unit imaginary quaternions all square to $$-1$$.
Meaning you can replace the complex number $$i$$ with the quaternions $$i$$,$$j$$, or $$k$$ and follow the same logic as the normal proof for Euler's forumla more or less. 

Using this representation we have
\\[
\frac{\partial}{\partial t}\tilde{q} = \frac{\partial}{\partial t}e^{\frac{\theta}{2}\vec{u}} = \frac{\partial}{\partial t}\left(\frac{\theta}{2} \vec{u}\right) e^{\frac{\theta}{2}\vec{u}} 
= \frac{\partial}{\partial t}\left(\frac{\theta}{2} \vec{u}\right)\tilde{q} 
\\]
The question is, what is $$ \frac{\partial}{\partial t}\left(\frac{\theta}{2} \vec{u}\right)$$?
The easiest way to answer this is to think about the role of $$\theta \vec{u} $$.
The rate at which it changes controls the rate at which the rotation changes. 
Meaning this quantity is in fact our angular velocity $$\vec{\omega}$$.
With this insight we can complete our equation
\\[
\frac{\partial}{\partial t}\tilde{q} = \frac{\tilde{\omega}}{2}\tilde{q} 
\\]
Where $$\tilde{w} = (0,\vec{w})$$.

So now we have an equation that defines how the orientation evolves over time in terms of the angular velocity. 
In future articles we will talk about how to descritize this equation in the same way we created the update rules for simple linear motion.

{% include links.html %}
