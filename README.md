# Docker-Notes
<hr>

**`docker run`**

# Docker Introduction

Docker is a containerization tool used to package an application along with all its dependencies so that it runs the same on any machine.

Normally when we build an application, it depends on many things like:
- code files
- libraries
- packages
- runtime software (Node, Python, OS, etc.)
- environment variables
- terminal commands to start the app

If we share only the code with another person, they may install different versions of dependencies or run different commands. Because of these differences, the application may fail to run. This is the common problem called:

“Works on my machine”.

Docker solves this problem by packaging the entire application environment together. So the application, dependencies, runtime, and required commands are bundled into one unit which can run anywhere Docker is installed.

## Example Problem

Suppose I build a web application using:
- code files
- libraries
- packages
- runtime software (Python / Node)
- terminal command to start the app

Now if I share the code with my friend, he may install:
- different versions of packages
- different Node or Python version
- wrong dependency names
- different OS environment
- different commands to run the project

Because of these differences the project may fail to run.

Docker solves this problem by **containerizing the entire application**.  
So the code, software, dependencies, and commands required to run the application are packaged together.

## Docker Architecture (Basic Flow)

In Docker we mainly work with three things:

1) Dockerfile  
2) Image  
3) Container

Flow of Docker:
project → Dockerfile → Image → Container

## Dockerfile

A Dockerfile is a configuration file where we define how the application environment should be created.

Inside the Dockerfile we write things like:
- which base software to use (Node, Python, etc.)
- create working folders inside the container
- copy project files into the image
- install required dependencies (npm install, pip install, etc.)
- expose application ports
- command to run when the container starts (npm start, python app.py, etc.)

So the Dockerfile basically describes how to build the environment needed to run the application.

## Docker Image

Using the Dockerfile, Docker builds a **Docker Image**.
The image contains everything needed to run the application:

The image contains:
- project code files
- runtime software (Node, Python, etc.)
- libraries and packages
- environment setup
- application start command

So an image is basically a **complete packaged snapshot of the project environment**.

You can think of it as an end-to-end packaged version of the application folder.  
Anyone who gets this image does not need to manually install software, dependencies, or configure anything.

Once the image is built, it can be shared with others.

## Docker Container

A container is a running instance of an image.

When we run an image using docker run, Docker creates a container from that image and starts the application.

The container behaves like a small isolated computer inside your system.  
It contains everything that was inside the image:
- project files
- software (Node / Python)
- libraries and packages

After the container is created, Docker runs the command defined in the Dockerfile (for example `npm start` or `python app.py`) and the application starts running.

Important points:
- multiple containers can be created from the same image
- each container runs independently
- containers are isolated environments
