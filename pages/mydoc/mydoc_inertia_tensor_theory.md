---
title: Inertia Tensor Theory
summary : "A summary of the theory of inertia tensors"
sidebar: mydoc_sidebar
permalink: mydoc_inertia_tensor_theory.html
folder: mydoc
toc : true
---

## Overview
Over the last 2 articles we laid out the ingredient list for rotational dynamics. 
Today we are going to tackle [Inertia Tensors](https://en.wikipedia.org/wiki/Moment_of_inertia) (also sometimes called moments of inertia, although they aren't quite the same as we will see below). 
As we said before the inertia tensor plays the role of mass for physical rotations.
Meaning it describes how resistant the object is to changes in angular position, just like inertial mass describes how resistant a point-like body is changes in position.

## Definition 
First we must accept a somewhat unfortunate fact (from a computational perspective).
Take a broom, stand it vertically, and spin it by rolling it between your thumb and index finger. 
Easy right?
Now take that same broom, hold it horizontally over your head, and try spinning it. 
It's much harder this time. 
Why is that? 
Well turns out that the moment of inertia depends on the axis you are spinning the object around. 

Given a system of $$N$$ particles about an origin $$\vec{O}$$ the moment of inertia is the sum of the squared distance of each particle to the rotation axis weighted by the particle mass.
That is: given a line $$L(t) = \vec{O} + t\vec{D}$$, with $$\vec{D} = (d_{1},d_{2},d_{3})$$ being unit length, the inertia tensor is
\\[
I_{L} = \sum\limits_{i=1}^{N}m_{i}(||\vec{r}\_{i}||^{2} - (\vec{D} \cdot \vec{r}_{i})^{2})
\\]
where $$\vec{r}_{i}$$ is the distance vector of partcle $$i$$ from the origin $$\vec{O}$$.

## Expansion
Now lets do some expanding
\\[
\begin{align}
I_{L} &= \sum\limits_{i=1}^{N}m_{i}(||\vec{r}\_{i}||^{2} - (\vec{D} \cdot \vec{r}_{i})^{2}) \\\
  &= d\_{1}^{2}\sum\limits\_{i=1}^{N}m\_{i}(y\_{i}^{2} + z\_{i}^{2}) + d\_{2}^{2}\sum\limits\_{i=1}^{N}m\_{i}(x\_{i}^{2} + z\_{i}^{2}) + d\_{3}^{2}\sum\limits\_{i=1}^{N}m\_{i}(x\_{i}^{2} + y\_{i}^{2})
\\\
  &- 2d\_{1}d\_{2}\sum\limits\_{i=1}^{N}m\_{i}x\_{i}y\_{i} - 2d\_{1}d\_{3}\sum\limits\_{i=1}^{N}m\_{i}x\_{i}z\_{i}- 2d\_{2}d\_{3}\sum\limits\_{i=1}^{N}m\_{i}y\_{i}z\_{i} \\\
  &= d\_{1}^{2}I\_{xx} +  d\_{2}^{2}I\_{yy} + d\_{3}^{2}I\_{zz} - 2d\_{1}d\_{2}I\_{xy} - 2d\_{1}d\_{3}I\_{xz} - 2d\_{2}d\_{3}I\_{yz}
\end{align} 
\\]

Looking at the defintion for the moment of inertia it shouldn't take much to convince you that $$I_{xx}$$ is the moment of inertia about the $$x$$ axis. 
Likewise for $$I_{yy}$$ and $$I_{zz}$$.
The cross terms $$I_{xy}$$, $$I_{xz}$$, $$I_{yz}$$ are sometimes called *products of inertia*

## Tensor form
The expansion above is most compactly written as a matrix equation
\\[
I\_{L} = 
\vec{D}^{T}
\begin{bmatrix}
I\_{xx} & -I\_{xy} & -I\_{xz} \\\
-I\_{xy} & I\_{yy} & -I\_{yz} \\\
-I\_{xz} & -I\_{yz} & I\_{zz} \\\
\end{bmatrix}
\vec{D}
=
\vec{D}^{T}J\vec{D}
\\]
The matrix $$J$$ is what is called the *inertia tensor* for the system.
It is made up of *moments of inertia*. 
$$J$$ is what we really care about when discussing motion of rigid bodies in terms of *Euler's second law*.
For a quick motivation as to why that is the case let's consider the example of a single particle. 
It's angular momentum, $$L$$, is written
\\[
\vec{L} = \vec{r} \times m\vec{v} = m(\vec{r} \times (\vec{w} \times \vec{r})) = m(||r||^{2}I - \vec{r}\vec{r}^{T})\vec{w}
\\]
Where we made use of $$\vec{v} = \vec{w} \times \vec{r}$$ and $$\vec{a} \times (\vec{b} \times \vec{c}) = \vec{b}(\vec{a} \cdot \vec{c}) - \vec{c}(\vec{a} \cdot \vec{b})$$
Now
\\[
m(||r||^{2}I - \vec{r}\vec{r}^{T}) =
m\begin{bmatrix}
y^{2} + z^{2} & -xy & -xz \\\
-xy & x^{2} + z^{2} & -yz \\\
-xz & -yz & x^{2} + y^{2}
\end{bmatrix}
\\]
You can see by comparing to our earlier derivations that this matrix is just the $$J$$ matrix but for a single particle. 
So
\\[
\vec{L} = J\vec{w}
\\]
which if we differentiate with respect to time gives us Euler's second law.

## Continous objects
Now in real life objects are made of neigh inummerable particles. 
Summing over all of them is intractable computationally. 
That is why we instead approximate real objects as solid objects containing infinite particles. 
In that case, the sums above get replaced by integrals over the region $$Q$$ occupied by the mass of the object.
\\[
I\_{xx} = \int\limits_{Q}(y^{2} + z^{2})dm
\\]
And so on for the other moments. 

If the mass is evenly distributed throughout the volume then $$dm$$ can be written in terms of the mass density $$dm = \rho dV$$ and the integral becomes a standard triple integral. 

## Coordinate systems
Up until this point I've avoided talking about coordinate systems on purpose.
But we can't ignore them any longer. 
There are 2 relevent coordinate systems we need to consider, model coordinates and world coordinates. 
It is often preferable to express $$J$$ in model coordinates.
There we have the capability to orient the object in such a way that $$J$$ is easy to compute.
In fact, as a real symetric matrix, there exists a coordinate system where $$J$$ is a diagonal matrix.
However we will be expressing *torque*, $$\vec{\tau}$$, and $$\vec{w}$$ in world coordinates so we need to know how to transfrom $$J$$ between coordinate systems.

First lets consider translations. 
Consider the translation $$x \rightarrow x+5$$. 
Well a simple [variable substition](https://en.wikipedia.org/wiki/Change_of_variables), $$u = x+5$$, returns all the integrals back to their original forms. 
So clearly translations don't affect $$J$$.
However, if we rotate our coordinate system by $$90$$ degrees such that $$y \rightarrow x, x \rightarrow -y$$ then clearly all the entries in $$J$$ pertaining to $$x$$ and $$y$$ will be swapped at the very least, if not sign flipped as well. 
From this we deduce that rotations can possibly affect $$J$$. 

So to transform $$J$$ from model to world coordinates we need to multiply it by the rotational part of the model matrix. 
Since the model matrix changes every time interval this means we'll have to perform this multiplication every cycle. 
However this is much better than recomputing $$J$$ from scratch every cycle.

It is possible to instead transport $$\vec{\tau}$$ and $$\vec{w}$$ to model coordinates. 
There we have the benefit of $$J$$ remaining a diagonal matrix. 
However since the model coordinate system is rotating with respect to the world coordinate system Euler's second law becomes a little more complicated. 
I may do an article about both ways to performing these calculations as I'm curious which is faster. 
More complicated inertia matrix or more complicated differential equation?

## Conclusion
Next time we'll tackle implementing this for our simple spheres and cubes.

{% include links.html %}
