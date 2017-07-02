---
title: Backward Euler (theory)
summary : "Learn about stability of numerical algorithms"
sidebar: mydoc_sidebar
permalink: mydoc_backward_euler_theory.html
folder: mydoc
toc : true
---

## Overview
Today we are going to learn about the last property of numerical algorithms we'll be talking about, **stability**. 
Like the other articles we'll be using a new integrator to highlight this topic.
However unlike other articles we'll actually be splitting this up over a few posts as there are few concepts to introduce.

## Stability
Last time we spoke about the accuracy of numerical methods, which was a measure of the difference between the exact solution and the approximate solution.
Stability is harder to define, but the idea is that certain problems are [stiff](https://en.wikipedia.org/wiki/Stiff_equation). 
Meaning there is something about the problem that causes wild inaccuracies when traditional methods are used to solve them. 
A good way to characterize a stiff problem is that it requries an excessively small time step $$h$$ in comparision to how smooth the exact solution is.

### A-stability
One way to investigate the stability of a numerical method is to apply it to a toy example where the exact answer is known. 
By performing this analysis for multiple methods you can get a feel how "stable" each one is to larger $$h$$ values.
In the case of [A-stability](https://en.wikipedia.org/wiki/Stiff_equation#A-stability) the toy problem is 
\\[
	\dot{y}(t) = ky(t)
\\]
\\[
	y(0) = 1
\\]
for any [complex number](https://en.wikipedia.org/wiki/Complex_number) $$k$$. 
The solution to this problem is $$y(t) = e^{kt}$$, which decays to $$0$$ as $$t \to \infty$$ if $$Re(k) < 0$$. 
$$Re(k)$$ being the real part of $$k$$.

### Stability function
For [Runge-Kutta](https://en.wikipedia.org/wiki/Runge%E2%80%93Kutta_method) methods (which all the methods we've dicussed so far are) we can write the update rule for $$y_{n}$$ as
\\[
y_{n+1} = \phi(hk)y_{n}
\\]
For example Explicit Euler 
\\[
y_{n+1} = y_{n} + h\dot{y}\_{n} = y_{n} + hky_{n} = (1 + hk)y_{n} = \phi(hk)y_{n}
\\]
We can expand on this inductively to say 
\\[
y_{n} = \phi(hk)^{n}y_{0}
\\]
Therefore the requirement that our solution decay to $$0$$ as $$t \to \infty$$ for $$Re(k) < 0$$ amounts to saying that $$|\phi(hk)| < 1$$.
Writing $$z = hk$$ we can define the **region of stabiltiy** for our numerical method to be the region where $$|\phi(z)| < 1$$.

For Explicit Euler the stability region is
\\[
|1 + z| < 1
\\]

which is plotted here in pink
<img src="./images/Backward Euler (Theory)/explict_euler_stability_region.svg" />

Of course if we wanted to use this approach to actually select a value of $$h$$ we'd have to know what $$k$$ is; 
but since this analysis technically only applies to the toy problem it's not really useful to use it in that way.
Instead we use the fact that the region of stability is small for this simple problem as intuition for how the method will behave on more complicted problems.
The lesson here being to keep your step size small if you are using Explicit Euler.

## Backward Euler
Now that we've introduced how stability is being measured we'll define our new integration scheme. 
[Backward Euler](https://en.wikipedia.org/wiki/Backward_Euler_method) (sometimes called Implicit Euler) is a new type of method we are going to learn about that is **L-stable**.
First lets lay down some defintions
{% include callout.html type="primary" content="**A-stable**: A method whose region of stability contains the entire left quadrant of the complex plane" %}
{% include callout.html type="primary" content="**L-stable**: A method that is A-stable and for which $$|\phi(z)| \to 0$$ as $$z \to \infty$$" %}
Put another way A-stable means that no matter which $$h$$ value you select the approximate solution will always decay to $$0$$ in the same way as the exact solution for the toy model.
L-stable is an additional restriction on A-stable that ensures a smoother decay to $$0$$.

To derive Backward Euler we are going to use a similar trick as we used for the velocity update rule in the [Verlet](mydoc_verlet.html#verlet_integration) article.
\\[
y_{n+1} = y_{n} + \int\limits_{t}^{t+h}\dot{y}(\tau)d\tau
\\]
We'll be approximating the integral on the right with the [rectangle rule](https://en.wikipedia.org/wiki/Rectangle_method) with a single rectangle.
That gets us
\\[
y_{n+1} = y_{n} + h\dot{y}\_{n+1} = y_{n} + hf(t_{n+1},y_{n+1})
\\]
I've been avoiding intruducing the *f* notation for as long as possible but I feel like this is the time.
The Runge-Kutta style methods often use the notation $$f(t,y) = \dot{y}(t)$$. It makes explicit the dependance of $$\dot{y}(t)$$ on $$t$$ and $$y$$.
Written in this way $$\dot{y}$$ is a function of $$2$$ formal parameters, $$t$$, and $$y$$. 

[Implicit equations](https://en.wikipedia.org/wiki/Implicit_function) express the left hand side in terms of functions of itself. 
For example the implicit equation $$x^{2} = x$$ has $$x$$ on both sides of the equal sign. 
Casting the problem as a root finding problem is a common strategy for solving these kinds of equations.
Continuing with our example instead of solving $$x^{2} = x$$ we could equivalently solve $$x^{2} - x = 0$$.

In the next article we will explore a general purpose root finding technqiue that will allow us solve the implicit equation that defines Backward Euler.
\\[
y_{n+1} - y_{n} - hf(t_{n+1},y_{n+1}) = 0
\\] 


{% include links.html %}
