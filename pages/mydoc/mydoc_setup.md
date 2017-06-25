---
title: Setup
summary : "Install requiste libraries and create a window"
sidebar: mydoc_sidebar
permalink: mydoc_setup.html
folder: mydoc
toc : false
---

## Overview
This will probably be the most frustrating part of of any of the tutorials. Setting up libraries never goes smoothly in my experience. However instead of trying to list all the possible ways things could go wrong I'm just going to take the easy route and rely on you to solve your own problems. There should be enough documentation on the web to figure out most any issue that might arrise.

I am developing this engine on Ubuntu so some of the specifics will be flavored for linux. However it should not be **that** hard to get this working on windows as well. Perhaps in the future I'll create a seperate page for that scenario.


I make use of the following libraries to aid in engine development:
* [GLFW](http://www.glfw.org/) - API for creating windows, reciving input events
* [GLEW](http://glew.sourceforge.net/) - Handles hooking up OpenGL function pointers
* [Eigen](http://eigen.tuxfamily.org/index.php?title=Main_Page) - General purpose linear algebra library
* [Doxygen](http://www.stack.nl/~dimitri/doxygen/) - Automated documentation generator

Some of you may be wondering why I am using Eigen instead of [glm](http://glm.g-truc.net/0.9.8/index.html). 
glm is tailored specifically towards the graphics community, so you think it'd be a perfect fit. 
However I've personally found its limitations to be annoying. For example it does not support matricies of arbitrary dimesnions. Eigen is a much more powerful library and I think it is worth the tradoff in having to implement some of the functionality that comes built into glm ourselves. 

## Code
If you've managed to setup all the libraries correctly you should be able to clone this [branch](https://github.com/AdamSturge/Engine/tree/blog_setup) and see an empty window. There is an included makefile so simply run 'make' at the root level to compile and run ./sim.o to execute the resulting binary. 

<img src="./images/Setup/empty_window.png" />

## What's ahead

Over the next few articles we will lay down the basis for our Engine. Unfortunately a lot goes into rendering even one triangle so you won't see much by way of visual progress until [Hello Sphere](mydoc_hello_sphere.html). But if you stick with me you'll learn some cool stuff I promise

{% include links.html %}
