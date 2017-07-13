---
title: Newtons Method
summary : "A general purpose root finding algorithm"
sidebar: mydoc_sidebar
permalink: mydoc_newtons_method.html
folder: mydoc
toc : true
---

## Overview
If you'll recall from [Backward Euler](mydoc_backward_euler_theory.html) we arrived at an update rule where $$y_{n+1}$$ relied on itself. 
This implicit method can be difficult to solve exactly, but by moving all the terms to the left hand side so that the right hand side is zero we can recast the problem as a root finding problem.
**Newton's Method** is one way to find the roots of complicated functions, as well shall see.

## Newton's method for functions of one variable
### The point-slope equation of a straight line
The point slope equation of a straight line is not quite as popular as the slope-intercept form so by way of refresher here it is.
\\[
f(x) - f(x_{1}) = m(x - x_{1})
\\]
It defines a straight line in terms of it's constant slope $$m$$ and a point on the line $$(x_{1},y_{1})$$.
We know some calculus so we can replace $$m$$ with the derivative $$f'$$.
Since the slope is constant we can evaluate the derivative at any point on the line, so a natural choice is $$x_{1}$$ 
\\[
f(x) - f(x_{1}) = f'(x_{1})(x - x_{1})
\\]

### Motivation
The motivation for Newton's method is that if we have an "okay" guess at the root, $$x_{i}$$, then we can use a straight line approximation to $$f(x)$$ at $$x_{i}$$ to improve our guess.
By repeating this process our guess can be iteratively improved to arbitrary accuracy. 

<a title="By Ralf Pfeifer (de:Image:NewtonIteration Ani.gif) [GFDL (http://www.gnu.org/copyleft/fdl.html) or CC-BY-SA-3.0 (http://creativecommons.org/licenses/by-sa/3.0/)], via Wikimedia Commons" href="https://commons.wikimedia.org/wiki/File%3ANewtonIteration_Ani.gif"><img width="256" alt="NewtonIteration Ani" src="https://upload.wikimedia.org/wikipedia/commons/e/e0/NewtonIteration_Ani.gif"/></a>

The update rule comes from setting $$f(x) = 0$$ in the point-slope form and solving for $$x$$
\\[
    f'(x_{i})(x_{i+1}-x_{i}) = -f(x_{i}) \iff x_{i+1} = x_{i} - \frac{f(x_{i})}{f'(x_{i})}
\\]

### Proof of convergence
This section will be kind of math heavy, but don't be too intimidated. 

Assume that the true root of $$f(x)$$ is $$a$$. 
Let $$x_{i}$$ be our current guess at $$a$$.
Then we can use [Taylor's theorem](https://en.wikipedia.org/wiki/Taylor%27s_theorem#Taylor.27s_theorem_in_one_real_variable) to expand $$f(a)$$ about $$x_{i}$$.
\\[
f(a) = f(x_{i}) + f'(x_{i})(a-x_{i}) + R
\\]
Where $$R$$ is the remainder term. 
There are multiple ways to characterize the remainder. We will be using the Lagrange form, which is based on the [Mean Value Theorem](https://en.wikipedia.org/wiki/Mean_value_theorem)
\\[
R = \frac{1}{2}f\'\'(c_{i})(a-x_{i})^{2}
\\]
where $$c_{i} \in (a,x_{i})$$ (or $$(x_{i},a)$$ depending on which is less than the other).

Since $$a$$ is the true root we know that the first equation is actually 
\\[
0 = f(x_{i}) + f'(x_{i})(a-x_{i}) + \frac{1}{2}f\'\'(c_{i})(a-x_{i})^{2}
\\] 

Dividing this whole thing by $$f'(x_{i})$$ and rearranging we have
\\[
\frac{f(x_{i})}{f'(x_{i})} - x_{i} + a = \frac{-f\'\'(c_{i})}{2f'(x_{i})}(a-x_{i})^{2}
\\]

Recall that our update rule is
\\[
x_{n+1} = x_{i} - \frac{f(x_{i})}{f'(x_{i})}
\\]

plugging this into our taylor expansion 
\\[
a - x_{n+1} = \frac{-f\'\'(c_{i})}{2f'(x_{i})}(a-x_{i})^{2}
\\]

Denote the error in our estimate by $$\varepsilon_{i} = a - x_{i}$$
\\[
\varepsilon_{i+1} = \frac{-f\'\'(c_{i})}{2f'(x_{i})}\varepsilon_{i}^{2}
\\]

so 

\\[
|\varepsilon_{i+1}| = \left\lvert\frac{f\'\'(c_{i})}{2f'(x_{i})}\right\rvert\varepsilon_{i}^{2}
\\]

What we want here is for $$|\varepsilon_{i+1}| \to 0$$ as $$i \to \infty$$. 
There are precise conditions for when this is gaurenteed to occur that I won't go into here. 
But suffice to say that if your initial guess $$x_{0}$$ is somewhat decent, and your function $$f$$ isn't too funky, then the error will converge to zero **quadratically**. 
Afterall $$\varepsilon_{i+1}$$ is proportional to $$\varepsilon_{i}^{2}$$.

## Newton's method for vector valued functions
Unfortunately for us we don't have a simple function of 1 variable. Recall that our update rule for Backward Euler is 
\\[
y_{n+1} - y_{n} - hf(t_{n+1},y_{n+1}) = 0
\\]
where $$y = (x,v) \in \mathbb{R}^{6}$$. 
Introducting some notation:
\\[
G(y_{n+1}) 
=
\begin{bmatrix}
x_{n+1} - x_{i} - hv_{n+1} \\\
v_{n+1} - v_{n} - h\frac{F(x_{n+1},v_{n+1})}{m}
\end{bmatrix}
=
\begin{bmatrix}
X(x_{n+1},v_{n+1}) \\\
V(x_{n+1},v_{n+1})
\end{bmatrix}
\\]
So we want to find the root, $$y_{n+1}$$, of the vector valued funciton $$G:\mathbb{R}^{2} \to \mathbb{R}^{2}$$.
To do this we will make use of the generalization of Newton's Method.
 
{% include note.html content="I am being a little more general here than usual by writing $$F(x_{n+1},v_{n+1})$$ instead of $$F(x_{n+1})$$. 
As I spoke about in [force generators](mydoc_force_generator.html) most of the forces we deal with only depend on position.
However for the purposes of this article we will allow for a velocity dependence as well. 
That way if we ever add forces that depend on velocity in the future we won't have to update our method" %}

At this point the subscript notation gets a little hairy. Let $$z_{i}$$ be the $$i^{th}$$ approximation to $$y_{n+1}$$. 
Then the generalized Newton's method is
\\[
\nabla G(z_{i}) (z_{i+1} - z_{i})= -G(z_{i})
\\]

$$\nabla G(z_{i})$$ is the [Jacobian matrix](https://en.wikipedia.org/wiki/Jacobian_matrix_and_determinant) of $$G$$ evaluated at $$z_{i}$$.

\\[
\nabla G =
\begin{bmatrix}
\frac{\partial X}{\partial x_{n+1}} & \frac{\partial X}{\partial v_{n+1}} \\\
\frac{\partial V}{\partial x_{n+1}} & \frac{\partial V}{\partial v_{n+1}} 
\end{bmatrix}
= 
\begin{bmatrix}
I & H \\\
-\frac{h}{m}\frac{\partial F}{\partial x_{n+1}} & I - \frac{h}{m}\frac{\partial F}{\partial v_{n+1}}
\end{bmatrix}
\\] 

Where $$I$$ is the $$3 \times 3$$ identity matrix, $$H$$ is $$hI$$, and the derivatives of the $$F$$ are $$3 \times 3 $$ Jacobians. 
\\[
\frac{\partial F}{\partial x_{n+1}}
=
\begin{bmatrix}
\frac{\partial F_{x}}{\partial x_{n+1,x}} & \frac{\partial F_{x}}{\partial x_{n+1,y}} & \frac{\partial F_{x}}{\partial x_{n+1,z}} \\\
\frac{\partial F_{y}}{\partial x_{n+1,x}} & \frac{\partial F_{y}}{\partial x_{n+1,y}} & \frac{\partial F_{y}}{\partial x_{n+1,z}} \\\  
\frac{\partial F_{z}}{\partial x_{n+1,x}} & \frac{\partial F_{z}}{\partial x_{n+1,y}} & \frac{\partial F_{z}}{\partial x_{n+1,z}}
\end{bmatrix}
\\]

\\[
\frac{\partial F}{\partial v_{n+1}}
=
\begin{bmatrix}
\frac{\partial F_{x}}{\partial v_{n+1,x}} & \frac{\partial F_{x}}{\partial v_{n+1,y}} & \frac{\partial F_{x}}{\partial v_{n+1,z}} \\\
\frac{\partial F_{y}}{\partial v_{n+1,x}} & \frac{\partial F_{y}}{\partial v_{n+1,y}} & \frac{\partial F_{y}}{\partial v_{n+1,z}} \\\  
\frac{\partial F_{z}}{\partial v_{n+1,x}} & \frac{\partial F_{z}}{\partial v_{n+1,y}} & \frac{\partial F_{z}}{\partial v_{n+1,z}}
\end{bmatrix}
\\]


Recall that our goal is $$z_{i+1}$$.
In the vector valued case Newton's method is a [system of linear equations](https://en.wikipedia.org/wiki/System_of_linear_equations) of the form $$A\delta = b$$.
If we can solve this system for $$\delta = z_{i+1} - z_{i}$$ then we can find $$z_{i+1}$$ by writing $$z_{i+1} = z_{i} + \delta$$.
Solving a system of linear equations is a well studied problem that I won't go into detail on. 
Safe to say the the Eigen library has many built in methods for solving this kind of problem, so we will be leveraging those. 

In the next article we'll finally implement Backward Euler
