---
title: Simple Shader
summary : "Create a simple shader for a scene"
sidebar: mydoc_sidebar
permalink: mydoc_simple_shader.html
folder: mydoc
toc : false
---

## Overview
My shader class is based off the shader described in [learnopengl](https://learnopengl.com/#!Getting-started/Shaders). Please read it. 

My code can be found in [shader.h](https://github.com/AdamSturge/Engine/blob/master/include/shader.h) and [shader.cpp](https://github.com/AdamSturge/Engine/blob/master/shader.cpp)

The Shader class handles reading and compiling shader files from disk. The actual shaders I use are below

## Vertex Shader
```c++
#version 330 core
layout (location = 0) in vec3 position;     
  
uniform mat4 model;
uniform mat4 view;
uniform mat4 projection;

void main()
{
    gl_Position = projection*view*model*vec4(position, 1.0f);
}    
```

This simple vertex shader handles translating a fragment from local coordinates to camera coordinates with projective perspective.

## Fragment Shader
```c++
#version 330 core
out vec4 color;

void main()
{
    color = vec4(1.0f,0.0f,0.0f,1.0f);
}

```
Whereas this simple fragment shader colors everything red.

{% include links.html %}
