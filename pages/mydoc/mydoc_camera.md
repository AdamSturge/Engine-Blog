---
title: Camera
summary : "Creating a camera for a scene"
sidebar: mydoc_sidebar
permalink: mydoc_camera.html
folder: mydoc
toc : false
---

## Overview
The camera I use is the camera described on [learnopengl](https://learnopengl.com/#!Getting-started/Camera). I won't repeat what's written there so I encourage you to read that article. 

If you don't want to read the details the long and short of it is that you can move the camera using the arrow keys. And if you uncomment this code in main.cpp you can look around with the mouse.

```c++
glfwSetCursorPosCallback(window,MouseCallback); 
glfwSetInputMode(window,GLFW_CURSOR, GLFW_CURSOR_DISABLED); 
```

For my implementation see [camera.h](https://github.com/AdamSturge/Engine/blob/master/include/camera.h) and [camera.cpp](https://github.com/AdamSturge/Engine/blob/master/camera.cpp)

{% include links.html %}
