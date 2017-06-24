---
title: Scene
summary : "Lets put everything we've learned together"
sidebar: mydoc_sidebar
permalink: mydoc_scene.html
folder: mydoc
toc : true
---

## Overview
We've done a fair bit at this point and our render loop is starting to loop a little messy. 
What's more the way we've been doing things so far will only get messier as we add more spheres to the scene. 
So it's time to introduce a container to wrap together everything we've discussed so far.

## Scene
We'll call this container Scene. It'll be a class that contains references to all the entities that are, well, in the scene. As well it will handle rendering and stepping the physics (by passing those requests down to other objects).

```c++
#include <physics_entity.h>
#include <model.h>
#include <memory>
#include <vector3G.h>
#include <time_integrator.h>
#include <shader.h>
#include <GLFW/glfw3.h>

#ifndef SCENE
#define SCENE
/**
    \brief A scene in the engine

    A scene contains (pointers to) a number of PhysicsEntity objects to simululate and a number of Model objects to render.
**/
class Scene
{
    private:    
        std::vector<std::shared_ptr<PhysicsEntity>> m_physics_entity_ptrs;
        std::vector<std::shared_ptr<Model>> m_model_ptrs;
        std::shared_ptr<TimeIntegrator> m_time_integrator;   
          
    public :
        /**
            Creates a Scene instance with default values for its TimeIntegrator and ForceGenerator members
        **/
        Scene();

        /**
            Creates a Scene instance with the provided TimeIntegrator and ForceGenerator members
            @param integrator TimeIngrator instance to handle time evolution of the PhysicsEntities in the scene
        **/
        Scene(std::shared_ptr<TimeIntegrator> integrator);
        
        /**
            Add a PhysicsEntity to the Scene.
            @param entity_ptr a shared pointer to the PhysicsEntity to be added to the scene
        **/
        void AddPhysicsEntity(std::shared_ptr<PhysicsEntity> entity_ptr);

        /**
            Loads the PhysicsEntity at the given index into the supplied shared pointer
            @param index index into the PhysicsEntity list for this Scene
            @param entity_ptr pointer that will be modified to point to the PhysicsEntity at the given index
        **/
        void GetPhysicsEntity(const GLuint index, std::shared_ptr<PhysicsEntity> &entity_ptr);

        /**
            Adds a Model to the scene
            @param model_ptr a shared pointer to the model to be added to the scene
        **/
        void AddModel(std::shared_ptr<Model> model_ptr);

        /**
            Loads the Model at the given index into the supplied shared pointer
            @param index index into the Model list for this Scene
            @param model_ptr pointer that will be modifed to point to the Model at the given index
            @param VAO uint that will be modifed to store the vertex array object id for the Model
        **/
        void GetModel(const GLuint index, std::shared_ptr<Model> &model_ptr, GLuint &VAO);

        /**
            @return The number of Physics Entites in the Scene
        **/
        GLuint GetPhysicsEntityCount();

        /**
            @return The number of Models in the Scene
        **/
        GLuint GetModelCount();

        /**
            Moves the physical simulation one time step forward
        **/
        void StepPhysics();
 
        /**
            Renders the Models in the Scene to the screen
        **/
        void Render(Shader shader);

        /**
            Removes all Model data from the GPU
        **/
        void CleanUp();
};
#endif
```

Whew boy that's a lot. Lets dig in.
 
First I'll draw your attention to the use of [shared_ptr](http://en.cppreference.com/w/cpp/memory/shared_ptr). Only suckers use raw pointers nowadays.
shared_ptr is a class that is used to allocate an object on the stack that tracks a pointer on the heap. That way when the stack variarble goes out of scope it can delete the pointer in its destructor. 
shared_ptr is used when you plan to be passing around pointers. When a shared_ptr is copied it tracks out many instances of itself are floating around. 
Only when the last instance is destroyed will the underlying pointer on the heap by deleted.

The first use of shared_ptr is in the vector lists of [Model](https://adamsturge.github.io/Engine/classModel.html) and [PhysicsEntity](https://adamsturge.github.io/Engine/classPhysicsEntity.html).
Although a [Sphere](https://adamsturge.github.io/Engine/classSphere.html) is both a Model and a PhysicsEntity, in general this will not be the case. 
Anyone who has played video games as encountered the infamous "invisable walls" that represent the boundries of the scene. 
These walls are physics entites in so far as they have collision physics, but clearly aren't being rendered. 
Another classic example is foliage that is being rendered but the player can seemlessly pass through.
The use of pointers here is primarily to avoid passing around copies of these possibly large entities. Put another way a scene merely tracks the entities, it doesn't own them.

The second use of shard_ptr is because [TimeIntegrator](https://adamsturge.github.io/Engine/classTimeIntegrator.html) is an abstract base class and cannot be constructed directly. 
If we were to try and store a variable of type TimeIntegrator we'd get compile time errors because we'd be implying the existance of a constructor.
So instead we have to store a pointer. 

With that out of the way the majority of the methods on Scene are self-explanitory. However some require a little more effort. 

### StepPhysics
Here's the implementation for *StepPhysics()*

```c++
void Scene::StepPhysics()
{
    for(std::shared_ptr<PhysicsEntity> entity_ptr : m_physics_entity_ptrs)
    {
        Vector3Gf xi = entity_ptr->GetPosition();
        Vector3Gf vi = entity_ptr->GetVelocity();
        GLfloat mass = entity_ptr->GetMass();

        Vector3Gf force;
        force.setZero();

        Vector3Gf xf;
        Vector3Gf vf;

        m_time_integrator->StepForward(xi,vi,mass,force,xf,vf);

        entity_ptr->SetNextPosition(xf);
        entity_ptr->SetNextVelocity(vf);

    }

    for(std::shared_ptr<PhysicsEntity> entity_ptr : m_physics_entity_ptrs)
    {
        entity_ptr->UpdateFromBuffers();
    }

};
```
This is just the code from the render loop wrapped in a for loop over all the physics entities in the scene.
However worth commenting on is how we first compute the next position and velocity for each entity and only then do we load the values from buffer.
The reason the code is split into 2 parts in this way is because otherwise our simulation would depend on the order the entites were added to the scene. 
For some forces, like say magnetic repulsion, the force acting on one entity depends on the position of another. 
In order to compute the proper force acting on each entity we have to "freeze" the sceen at the current time step, compute all the forces, and only then use them to update the positions and velocities.
Now for this particular case we are setting the force to be zero so it wouldn't matter but in the future that won't be the case.

### Render
Onto *Render(Shader shader)*

```c++
void Scene::Render(Shader shader)
{
    GLint modelLoc = glGetUniformLocation(shader.Program, "model");

    const GLuint model_count = GetModelCount();
    std::shared_ptr<Model> model_ptr;
    for(int i = 0; i < model_count; ++i)
    {
        GLuint VAO;

        GetModel(i,model_ptr,VAO);

        Eigen::Matrix<float,4,4> model_matrix = model_ptr->GetModelMatrix();
        glUniformMatrix4fv(modelLoc, 1, GL_FALSE, model_matrix.data());

        glBindVertexArray(VAO);

        glDrawElements(GL_TRIANGLES, 2*model_ptr->GetMesh().GetNumEdges(), GL_UNSIGNED_INT,0);

        glBindVertexArray(0);

    }
}

```  
There isn't a lot worth commenting on here. We loop over all the models, load them one at a time from the scene, and draw them to the screen.

### CleanUp
And finally *CleanUp*
```c++
void Scene::CleanUp()
{
    const GLuint model_count = GetModelCount();
    std::shared_ptr<Model> model_ptr;
    for(int i = 0; i < model_count; ++i)
    {
        GLuint VAO;
        GetModel(i,model_ptr,VAO);
        model_ptr->GetMesh().CleanUp();
    }
}
```
This method handles removing the mesh data from the GPU for each model. 

## Render loop
We'll need to make some pretty significant changes to our render loop to incorperate our new Scene class. Here's the full thing.

```c++
    // Create a scene using Explicit Euler for time integration
    std::shared_ptr<TimeIntegrator> time_integrator_ptr(new ExplicitEuler(0.01f));
    Scene scene(time_integrator_ptr);

    // Create a sphere that will move up and to the right
    std::shared_ptr<Sphere> sphere1_ptr(new Sphere(1.0f, Vector3Gf(0.0f,0.0f,0.0f), Vector3Gf(10.0f,15.0f,0.0f), 1.0f));
    scene.AddPhysicsEntity(sphere1_ptr);
    scene.AddModel(sphere1_ptr);

    // Create a sphere that will move up and to the left
    std::shared_ptr<Sphere> sphere2_ptr(new Sphere(1.0f, Vector3Gf(0.0f,0.0f,0.0f), Vector3Gf(-10.0f,15.0f,0.0f), 1.0f));
    scene.AddPhysicsEntity(sphere2_ptr);
    scene.AddModel(sphere2_ptr);

    bool start = false; // true is physics is runnings
    while(!glfwWindowShouldClose(window))
    {
        GLfloat current_frame = glfwGetTime();
        delta_time = current_frame - last_frame;
        last_frame = current_frame;
    
        glfwPollEvents();
        DoMovement();   

        if(keys[GLFW_KEY_F])
        {
            glPolygonMode( GL_FRONT_AND_BACK, GL_LINE );
        }

        if(keys[GLFW_KEY_G])
        {
            glPolygonMode( GL_FRONT_AND_BACK, GL_FILL ); 
        }

        // Press p to start/stop the physics simulation
        if(keys[GLFW_KEY_P])
        {
            start = !start;
        }

        glClear(GL_COLOR_BUFFER_BIT);     

        glEnable(GL_DEPTH_TEST);
        glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);
    
        shader.Use();

        Eigen::Matrix<GLfloat,4,4> view;
        view = camera.GetViewMatrix();
        
        Eigen::Matrix<GLfloat,4,4> projection;
        projection = perspective<GLfloat>(camera.m_zoom, ((GLfloat)WIDTH)/((GLfloat)HEIGHT), 0.1f, 100.0f);          
      
        GLint modelLoc = glGetUniformLocation(shader.Program, "model");
        GLint viewLoc = glGetUniformLocation(shader.Program, "view"); 
        GLint projectionLoc = glGetUniformLocation(shader.Program, "projection");

        glUniformMatrix4fv(viewLoc, 1, GL_FALSE, view.data());
        glUniformMatrix4fv(projectionLoc, 1, GL_FALSE, projection.data());
  
        // Step the scene forward in time
        if(start)
        {
            scene.StepPhysics();
        }        
       
        // Render the scene
        scene.Render(shader);

        glfwSwapBuffers(window);
        glBindVertexArray(0);
    }    
    
    // Remove mesh(es) from GPU
    scene.CleanUp();

    glfwTerminate();
```
In addition to adding the Scene class this loop now had a "play/pause" button. Press 'p' to toggle the physics simulation between on and off. 

If everything works you should be greeted with the following

<video controls>
    <source src="./images/Scene/two_spheres.mp4" />
</video>

Here is the full source [code](https://github.com/AdamSturge/Engine/tree/blog_scene)

{% include links.html %}
