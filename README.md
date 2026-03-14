# Docker Introduction

Docker is a containerization tool used to package an application along with all its dependencies so that it runs the same on any machine.
**`
Normally when we build an application, it depends on many things like:
- code files
- libraries
- packages
- runtime software (Node, Python, OS, etc.)
- environment variables
- terminal commands to start the app
`**
If we share only the code with another person, they may install different versions of dependencies or run different commands. Because of these differences, the application may fail to run. This is the common problem called:

“Works on my machine”.

Docker solves this problem by packaging the entire application environment together. So the application, dependencies, runtime, and required commands are bundled into one unit which can run anywhere Docker is installed.

### Example Problem

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

## Docker Architecture (Flow)

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

After the container is created, Docker runs the command defined in the Dockerfile (for example **`npm start` or **`python app.py`) and the application starts running.

Important points:
- multiple containers can be created from the same image
- each container runs independently
- containers are isolated environments

### Example

Suppose you built a web application using:

- code files
- Node.js runtime
- npm packages
- specific dependency versions
- a command like npm start

Instead of sharing only the code, Docker allows you to package everything into an image.

Your friend only needs to:

install Docker  
pull or build the image  
run the container  

The application will run exactly the same on their machine without manually installing dependencies.

## Simple Architecture Summary

Project Code
     ↓
Dockerfile (instructions to build environment)
     ↓
Docker Image (packaged application environment)
     ↓
Docker Container (running application)

<hr>

# React Project with Docker

Step 1 — Create React project

npx create-react-app testapp  
cd testapp  
npm start  

Project runs locally.

### node_modules

node_modules contains all packages required to run the project.
We usually do NOT include node_modules while building Docker image because Docker installs dependencies itself.
So we can delete it:
rm -rf node_modules
If needed reinstall:
npm install

## Dockerfile

Create a file named Dockerfile and write:
**`
FROM node:20-alpine
WORKDIR /myapp
COPY . .
RUN npm install
EXPOSE 3000
CMD ["npm","start"]

<hr>

# dockerfile full template 

Dockerfile is used to define how a Docker image should be built.
Each instruction creates a new layer in the image.
follow the below Precidence:

## 1) Base Image
This is the starting environment of the container.
Usually contains OS + runtime software.

Examples:
**`FROM node:20-alpine
FROM python:3.11
FROM ubuntu:22.04
FROM tensorflow/tensorflow:latest

Explanation:
**`node:20-alpine → Node runtime with Alpine Linux (lightweight).

## 2) Working Directory
Creates a folder inside the container and sets it as the working directory.

**`WORKDIR /myapp

Meaning:
All future commands will run inside this folder.

## 3) Copy <dependency files> for dependencies installation
we do this before RUN , since npm install requires **`package.json file & **`requirements.txt required before doing **`RUN pip install -r requirements.txt

COPY source destination

Examples:
**`COPY package*.json .     → package.json and package-lock.json used for npm install
**`COPY requirements.txt .  → required for pip install -r requirements.txt

For other setups:
**`COPY pyproject.toml .
COPY Pipfile .
COPY pom.xml .
COPY go.mod .
etc

These files are copied first so that dependency installation can run using RUN.

## 4) RUN
Used to execute terminal commands during image building.
so that we can install dependencies required by the application.

Examples:
**`RUN npm install
RUN pip install package
RUN pip install -r requirements.txt
RUN apt-get update
RUN apt-get install -y package_name
RUN apt update && apt install -y curl
RUN venv venv
etc

## 5) Copy Files
Used to move project files from host → container image.
After dependencies are installed, copy the rest of the project source code.

**`COPY source destination

Example: -> Copy all project files from host to /myapp inside the container.
**`COPY . . 

Example: -> specific files copy
**`COPY project/python.py .
COPY project/service.py .
COPY requirements.txt .
etc

Note: If application code changes, Docker will only rebuild this step instead of reinstalling dependencies. thus faster image building next time.

## 6) Port
Expose tells Docker which port the application inside container will use.

**`EXPOSE port

**`EXPOSE 3000  -> Application inside container runs on port 3000.

Note:
since containers are isolated. so the port is not accessible outside automatically.
so we do port mapping -> **` docker -p host_port:container_port Image -> **`docker run -p 3000:3000 myimage
also for specific applications like flask, fastapi, streamlit, etc we have to set the host="0.0.0.0" , instead of localhost or etc.

## 7) Command
Defines the command that will run when the container starts.

Examples:
**`CMD ["npm","start"]
CMD ["python","app.py"]
CMD ["node","server.js"]
CMD ["uvicorn","main:app","--host","0.0.0.0","--port","8000"]

Important:
CMD can be overridden using **`docker run [COMMAND].

## Example Node project: 

**`FROM node:20-alpine
WORKDIR /myapp
COPY package*.json ./     # do this before RUN , since npm install requires package.json file
RUN npm install
COPY . .                  # we do this after RUN and not before ... since better and faster.
EXPOSE 3000
CMD ["npm","start"]

## python project example: 

**`FROM python:3.11
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
EXPOSE 8000
CMD ["python","app.py"]


xtras:
**`ADD file.tar.gz /app        → copies files like COPY but also auto-extracts archives and supports URLs.
ENTRYPOINT ["python","app.py"] → sets the main command that always runs when the container starts.
ENV NODE_ENV=production     → defines environment variables inside the container.
ARG NODE_VERSION=20         → defines a build-time variable usable during docker build.
VOLUME /data                → declares a mount point for persistent container data.
LABEL version="1.0"         → adds metadata information to the image.
LABEL author="ujwal"        → stores author or other descriptive metadata in the image.

----------------------------------------------------------------------------------------------------------------------------------
# Docker Images

After writing the Dockerfile, we create an image using the docker build command.

Inside the project folder: **`$testapp

**`docker build . → builds an image but without a name (only image ID).

docker build -t name:tag . → builds an image with a specific name and tag. (only lowercase name ,tag)
ex: docker build -t react_webapp:version_1 .

## Viewing Images

docker image ls → shows all Docker images present in your system (which has name:tag)

docker image ls -a → shows all images including untagged images with no name:tag (<none>:<none>).

## Deleting Images

docker rmi image_id/name:tag

You cannot delete an image if a container is using it.
First delete the container, then delete the image.

## Inspect Image

docker image inspect image_name:tag → shows detailed information about the image.

## Image Tag Reuse (Important)

If you rebuild an image using the same name:tag, Docker does not delete the old image.

Example:

docker build -t react_app:1 .

Later after updating code:

docker build -t react_app:1 .

What happens:
react_app:1 → now points to the new image  
old image → becomes <none>:<none>

So the tag moves to the new image, but the old image still exists internally.

you will NOT be able to see the image when you do -> docker image ls -a  , but image still there.

Also if a container was created from the old image, it will continue running with the old code. (this proves image is still there)

## When using docker build .

If we build without specifying a name.
docker build .
Docker creates an image with only an image ID (no name or tag).

If we run it again after code changes:
docker build .
A completely NEW unnamed image is created with new Image_id but No image:tag , and the previous one is NOT deleted.

So if you do -> docker image ls -a  then you will be able to see all those images.

(so its NOT like the above where you rebuild with same name:tag, where the images were getting disappear , here Images dosent disappear)

## Default Tag

If we specify only the image name without tag then Docker automatically assigns the tag -> latest

docker build -t project .  -> project:latest

and then if you again rebuild:
docker build -t project
then agian it will give the same poject:latest name . and thus the Image Tag Reuse will happen and then old image will exist but disappear.

## Where Images Are Stored

Docker images are not stored inside the project folder.  
All images created on the system are stored inside Docker's internal storage (managed by Docker).

We can access and manage them using:
- Docker CLI
- Docker Desktop application

## Cleaning Unused Images

docker system prune -a  → removes unused containers, dangling images, and unused build cache to free disk space.

----------------------------------------------------------------------------
# Docker Containers

After building an image, we can create multiple containers from that image.

##Creating Containers

docker run image_id/image_name:tag    (You can find the image ID/name:tag using: docker image ls -a)

## What Happens When We Run a Container

When we run:
docker run image_id/name:tag

Docker does two things internally:
docker create 
docker start container
So `docker run` is basically create + start combined.

## Container Execution Behavior 

A container runs whatever command is defined in the image (CMD or ENTRYPOINT).

Example:
CMD ["npm","start"]

If the program runs continuously (web server), the container will keep running.

If the program finishes execution (like printing output and exiting), the container will automatically stop.

## Stopping Containers

If the container is running in foreground:-

Ctrl + C

If the container is running in Background:-

docker ps → shows running containers
docker stop container_id/container_name → stops the container

## Starting an Existing Container

Do NOT run:
docker run image
because that will create a new container.

Instead start the existing one:
docker ps -a → shows all containers (running + stopped)
docker start container_id → Start a container

## Viewing Containers

docker ps → show running containers

docker ps -a → show all containers (running + stopped)

## Removing Containers

docker rm container_id/container_name → to remove a container

docker rm -f container_name → Force remove a running container

## Run Container and Auto Delete

docker run --rm image_name → This will create a container, run it, and automatically delete it when the container stops. (create container, run it, delete container when it stops)

## Viewing Logs

docker logs container_name → Shows logs and output of the container.

## Restart Container

docker restart container_name

## Open Shell Inside Container

docker exec -it container_name sh  → This opens an interactive shell inside the running container.

## Port Binding (Connecting Container to Browser)

When a container runs an application (example: React app using `npm start`), it runs inside the container environment. Even if the app is running on port 3000 inside the container, we still cannot access it from our browser.

This is because containers are isolated from the host system. So we must explicitly connect the container port to a port on the host system.
This is done using **port binding**.

Syntax:
docker run -p host_port:container_port image_id

Example:
docker run -p 3000:3000 image_id

Meaning:
system port 3000 → container port 3000

Important Concept

There are two things involved:
1) The port where the application is actually running inside the container.
2) The port on the host system that connects to that container port.

So if the React app runs on port 3000 inside the container, the container port must be 3000.  
The system port can be any available port like 3000, 3001, 3002, etc.

Example:
docker run -p 3000/3123/3069:3000 image_id

## Multiple Containers with Same Internal Port

Containers are isolated environments, so multiple containers can run applications on the same internal port (like 3000).
But the host system is not isolated and can only assign one application per port, so each container must use a different host port.
so container_port can be same of mulitple containers, but the Host_post must be different. 

Example commands:
docker run -d -p 3000:3000 image_id   // container 1
docker run -d -p 3001:3000 image_id   // container 2
docker run -d -p 3210:3000 image_id   // container 3

## Running Container in Background

When we run a container normally, it occupies the terminal.

docker run -d image_name → To run it in background:

Example:
docker run -d -p 3000:3000 image_name

This starts the container in detached mode.
so now we can run other cmds in the terminal. 

but it only works with (run) and not with (start) -> docker start -d container_id/name , so you cannot run a existing container in the background

## Container Naming

Docker automatically assigns random names to containers.

We can give a custom name:
docker run --name container_name image_name:tag/id

Example:
docker run --name react_app image_name

when making a container it gives a random name to the container ... so instead we can give name by ourself
docker run -d --rm --name "Container1_reactapp" -p 3000:3000 image_id/image_name:tag   

## Interactive Container

To make a Container with interactive termial (basically we can take input from user in the terminal)
Used when the container program needs user input.

docker run -it image_id/name:tag    (it -> interactive terminal)

## Combine Multiple Options

We can Combine Multiple options in a single cmd.

Example:
docker run -d --rm --name react_container -p 3000:3000 image_name:tag

Meaning:
- run in background
- auto delete container after stop
- assign custom name
- map ports

----------------------------------------------------------------------------------------------------------------------------------
# docker run cmd variations 

syntax:
docker run [OPTIONS] image_id/image_name:tag [COMMAND]

-------[OPTIONS]--------
You can write mulitple Options & There is NO Precedence.
but they should be before the image_name.

empty → create a container from the image and start it in the foreground (terminal attached).
docker run image_id/name:tag

-d → run container in detached mode (background).
docker run -d myimage:v1

--rm → container will be automatically deleted when it stops.
docker run --rm myimage:v1

--name container_name → run container with a custom name instead of auto-generated name.
docker run --name mycontainer myimage:v1

-p host_port:container_port → map host port to container port so application is accessible outside container.
docker run -p 3000:3000 myimage:v1

-it → run container in interactive terminal mode. (it -> interactive terminal)
docker run -it ubuntu:latest

-v volume_name:container_path → mount docker volume to container for persistent storage. container_path -> workingdir/..
docker run -v myvolume:/my_app/data myimage:v1

-v host_repo_path:container_path → bind mount host folder to container folder. container_path -> workingdir/..
docker run -v C:\project:/my_app myimage:v1

-e VARIABLE=value → pass environment variable to container.
docker run -e ENV=production myimage:v1

--network network_name → run container connected to a specific docker network.
docker run --network my-net myimage:v1

-------[COMMAND]--------

- You can write the command you want to run after the container starts.
- It overrides the default CMD written in the Dockerfile.
- write it after the image_name
- (write the cmd just like u write in terminal, with space seperated)

ex: 
docker run image_name npm run dev
docker run --name conatiner1 image_name python file.py

----------------------------------------------------------------------------------------------------------------------------------
# XTRA: Docker Image Behavior Experiment (Retag)

Experiment performed to understand how Docker handles images and containers.

Created an image and container:
docker build -t new:1 .
docker run -d --name container1 -p 3001:3000 new:1

Updated the code and built the image again with the SAME name:tag , and run the container again.
docker build -t new:1 .
docker run -d --name container2 -p 3002:3000 new:1

Again updated the code and rebuilt:
docker build -t new:1 .
docker run -d --name container3 -p 3003:3000 new:1

Observation:-

Each time we rebuild using the same name:tag (new:1):
- Docker creates a new image with updated code.
- The tag new:1 moves to the newest image.
- The older images lose their tag and become <none>:<none>.
- Containers created earlier still keep using the image they were created from.

So the images are NOT overwritten.  
New images are created and the tag simply moves to the latest one.
And the older images just lose their name:tag and become <nono>:<none> and disappaer , but still exist.

Result:

container1 → running old code (old image)  
container2 → running updated code  
container3 → running newest code

Delete latest container and image:

docker stop container3
docker rm container3
docker rmi new:1

Observation:

Now when running:
docker image ls -a
No image was shown with name new:1.
But container1 and container2 were still running.

Explanation:
Containers internally keep a reference to the image they were created from.  
Even if the image tag is removed, the container can still run because the underlying image layers still exist in Docker storage.

Delete remaining containers:

When container1 and container2 were deleted, then the old image refernce Images are also deleted. 
and Disk becomes Empty (this tells that those images were also remove as we removed those conatiners)

Final Understanding:

- Docker does not overwrite images when rebuilding with the same tag.
- The tag moves to the new image.
- Old images remain internally while containers reference them.
- Once all containers using those images are deleted, Docker removes the unused image layers automatically.

----------------------------------------------------------------------------------------------------------------------------------
# Pre-Defined Images:

Many images are already pre-built by the community or companies. Instead of creating everything from scratch, we can simply download these ready-made images from the internet and use them to run containers.

Docker provides a public registry called **Docker Hub**, which is similar to GitHub. Just like we push and pull code repositories on GitHub, we can push and pull **Docker images** from Docker Hub. This allows developers to share images and reuse common software environments.

docker pull image_name:tag
docker image ls  -> u can see the image 
docker run imageid/name:tag -> to make containers

Example:
docker pull nginx
docker image ls
docker run -d --rm --name "ngnix" -p 8080:80 ngnix:latest 

----------------------------------------------------------------------------
# Docker Image Versioning and Container Deployment 

When we update our application code and want to deploy the new version, we do NOT need to create the Dockerfile again.  
A Dockerfile usually needs to be written only once. It only needs to be updated if we add new dependencies, software, packages, or change startup commands.

After updating the code, we simply build a **new Docker image** from the same Dockerfile.

docker build -t image_name:tag .

This new image will contain the updated code.  
We can then create containers from this new image and run the updated version of the application.

Important:
Docker images are **immutable**, meaning once an image is created it cannot be modified.  
So we never update an existing image by putting new code inside it. Instead we build a **new image**.

However we can reuse the same name:tag if needed (Docker will move the tag to the newest image).

# Deployment Concept

Images are usually created when the code is ready to be deployed.

We do NOT build images every time we save code or add a small change during development (for that we use Git/GitHub).

Instead, images represent **deployable versions** of the application.

Example:
image → website:version01
image → website:version02
image → website:version03

Each image represents a new version of the application.

We Dont Give a new Image Name For each deploy versoin, instead we use Tag_name
Tags are used to represent versions of the image.

Example:
image → website:version1
update code  
image → website:version2
update code  
image → website:version3

So only make Image When you want to Deploy. 
(dont make after adding a feature or etc , Only when want to Deploy or share)

NOTE:
so images you can think like this -> you made a basic website then made a image and deployed , and then updated some stuff then again made a new image and then deployed the new version, and removed the previous deployed version.
or you can also think like -> you made different verions of ur website and deployed them all at the same time, to differnt users/locations/etc (ex:- 200 differnt versin of facebook runs parallely .. to test differnt features and stuff)

## Containers and Scaling

From a single image we can create many containers.

Containers are used for horizontal scaling.

Instead of increasing RAM or CPU of one server (vertical scaling), we run multiple containers of the same application across multiple servers.

Example:

container1 → running website  
container2 → running website  
container3 → running website  

All these containers run the same code from the same image.

These containers are usually connected to a load balancer.

Flow:

User Request → Load Balancer → Available Container

The load balancer forwards requests to the container with the lowest load.

----------------------------------------------------------------------------------------------------------------------------------
# general docker cmds 

docker --version
docker info
docker system df

docker system prune → deletes stopped containers, unused networks, dangling images, and build cache.
docker system prune -a → deletes everything unused: stopped containers, unused networks, ALL images not used by any container, and build cache.

(if there are no img , container the memory used will be 0)

start docker engine -> open docker desktop
stop docker enginer -> quit docker desktop  /or/ wsl --shutdown (this will shutdown the wsl, open docker desktop to start the docker engine agian)

----------------------------------------------------------------------------------------------------------------------------------
# docker hub :

Docker Hub is a cloud registry used to store and share Docker images. 
It works similar to GitHub, but instead of storing code repositories, it stores Docker images.

- Unlimited public repositories
- 1 private repository (free plan)

A repository on Docker Hub mainly contains:
- Docker images
- Different versions of the image using tags

1) Create a Repository on Docker Hub using the web interface.

2) Connect docker_hub and docker_engine
docker login 

3) Connect the local_Image with docker_hub 
To push an image, the image name must match the Docker Hub repository name.

So Either build a new Image with the same Repo Name:

docker build -t dockerhub_user_name/same_repo_name:any_tag_name .
ex: docker build -t ujwalsahu/demo-webapp:1 .    

OR Rename a already existing Image:

docker tag old_name:tag new_name:tag  -> rename the image to repo name
ex : docker tag webapp:1 ujwalsahu/demo-webapp:1

4) Push the Image
docker push dockerhub_user_name/repo_name:tag_name -> push local_images on docker-hub (particular repo)
ex: docker push ujwalsahu/demo-webapp:1  

5) Pushing New Versions
If the application code changes, build a new image with a new tag.

Example:
docker build -t ujwalsahu/demo-webapp:01 .
docker push ujwalsahu/demo-webapp:01
update code
docker build -t ujwalsahu/demo-webapp:02 .
docker push ujwalsahu/demo-webapp:02

Important rule:
So basically in a dockerhub repo all images must have -> the same image_name but tag_name can be differnt.
and so locally we have to make the images with the -> same_image_name_as_repo : any_tag_name.   and only then it will push to docker hub.

6) Pull Image from Docker Hub
we can clone any image from docker hub, and that images are stored into our docker desktop application folder in c:drive, and using cli or desktop app we can access those images and make containers using it.

Dockerhub > repo > tag > copy the name  

docker pull name/repo:tag  -> clones that specific tag ka image of that repo
docker pull name/repo  -> clones the latest image of that repo

docker run image_id/name:tag -> Then run the image and make container

// like dockerhub there are other registeries also like AWS-ECR (elastic container registery) , Google-GCR , Azure-ACR 
// so over there also we can push and pull our images .

----------------------------------------------------------------------------------------------------------------------------------
# Docker Volume

Docker containers are isolated and temporary. If a container stores data inside its own filesystem, that data exists only inside that container. If the container is deleted, the data stored inside it is also deleted.

Example:

Suppose we have a program that takes names from users, stores them in a file (names.txt), and prints all stored names.

Container1 → user enters 3 names → file contains [A, B, C]
Container2 → if we run the same program again, the file will be empty because it is a different container.

Also if Container1 is deleted, the stored names are lost.

To solve this problem, Docker provides Volumes.
A Docker Volume is a persistent storage location managed by Docker. Instead of storing data inside the container, the container stores data in the volume. The volume exists outside the container, so even if the container is deleted, the data remains safe.

Docker Volumes provide persistent storage outside the container filesystem.

So data:
container → writes data → volume
Instead of:
container → writes data → container filesystem

## Example:

container filesystem
   ├── app
   ├── node_modules
   └── data.txt

If container is deleted then data.txt ❌ gone

volume
   └── data.txt
container
   ├── app
   ├── node_modules
   └── mounted path → volume

Now if container deleted volume still exists thus data safe

## volume cmds

docker volume create volume_name -> Creating a Volume

docker volume ls -> List volumes

docker volume inspect volume_name -> Inspect volume

docker volume rm volume_name -> Delete volume

docker volume --help -> to see the volume cmds

## Using Volume with Container

docker run -v volume_name:container_path image   -> so any data inside the container_path will be stored in that volume

Example:
docker run -v myvolume:/app/data image_name -> Anything written inside /app/data will be stored in the volume instead of the container filesystem.

Other Method — Docker auto-creates volume if not exist
docker run -v myvol:/app/data image -> Here If myvol does not exist, Docker creates it automatically. So most people directly use -v.

## Example with Multiple Containers

Container1:
docker run -v myvolume:/app/data image
User enters:
A B C
Stored in volume.

Now start Container2:
docker run -v myvolume:/app/data image
Program prints:
A B C
Because both containers share the same volume.

Even After Container Deletion
container1 deleted
container2 deleted

Volume still exists:
myvolume → [A, B, C]

## Key Concept

Containers should ideally be Stateless, meaning they should not permanently store important data.

The container's job is only to run the application. Containers may be created, deleted, or scaled many times, so storing important data inside them is not reliable.

So persistent data should be stored in:
- Docker Volumes
- Databases
- External storage systems
Thus Stateless containers can be used for Horizontal Scaling, but Stateful cannot.

## Multiple Volumes Example

Image1_Container1 uses volume1 → stores 3 names
Image1_Container2 uses volume2 → stores 5 names
Even though both containers use the same image, their data will be different because they are connected to different volumes.

## Meaning of -v volume_name:container_path

syntax:
docker run -v volume_name:container_path image 
Here,
volume_name ->  the Docker volume where data will be stored
container_path -> folder inside container whose data will be stored in that volume

Everything inside a container is stored in the WORKDIR Folder, so in Container_path we always have to mention it.

Example Dockerfile: WORKDIR myapp

/myapp
 ├── data
 ├── logs
 └── files

docker run -v myvol:/myapp image -> thus all the data stuff in the myapp folder will be stored in volume
docker run -v myvol:/myapp/data image -> thus all the data stuff in the myapp/data folder will be stored in volume
docker run -v myvol:/myapp/folder1/data image -> thus all the data stuff in the myapp/folder1/data will be stored in volume

(Also, only the data part is stored in the volume, not things like code files or application files. Basically only the user-generated or runtime data is stored there. Docker handles this automatically — we do not have to explicitly specify that "store data.txt" or any particular file.)
(And this data is not stored where the image is stored. It has nothing to do with the image storage. The data is stored separately in a proper Docker volume.)

----------------------------------------------------------------------------------------------------------------------------------
# docker bind mounts

In Docker volumes, the container data is stored in Docker’s internal storage and remains safe even if the container is deleted. However, this data is hidden inside Docker’s volume storage, so it is not easy to directly view or edit those files from the host machine.

so while coding and testing ... we needt to again and again test the code and data... to check if it works on this data or not.
so in that case you have to again and again make new images and run it to test if the new code and data is working. 
so to solve this we have docker bind mounts ... which is basically ... when we create a container then we connect it with our local repo ... 
so bascillay the container to directly use a folders and files from our local repo of host machine.
so that if we change any code , data in our repo files then that changes will also be reflected in the container ... so 
no need to agian and agian create images and run ... we just simply make a bind mount container ... and do testing using just single container ... 

## Example
suppose you are developing a node / react / python app.

you write code in your computer:
C:\project
 ├─ index.js
 ├─ package.json

but if you run docker normally - docker run image_name
then the container has its own filesystem and cannot see your host files.
so if you update code on your computer, the container will not see those changes.
you would have to rebuild the image again and again:
docker build
docker run
this is slow during development.

so to solve this problem we use bind mounts.
bind mount connects:
host folder  ↔  container folder
so the container directly uses the host filesystem.

## example

C:\project
 ├─ index.js
 ├─ package.json

/app <-connected-> C:\project 
 ├─ index.js
 ├─ package.json

now container reads files directly from your computer.
so if you edit code in -> C:\project\index.js
the container immediately sees the change.

syntax:
docker run -v Host_repo_path:container_WORKDIR_path image

Host_repo_path → where files exist on your PC
container path → the folder in which the files u want to get updated 

if entire repo you want to bind then:
repo_folder:working_dir_folder  -> ex ->  C:\names_app:/app

if particuale files only u want to bind mount then:
repo_folder:sub_folder_of_workingdir -> ex -> C:\names_app:/app/folder1/data
(so only the data folder inside app/folder1 will be bind mounted.)

we can also just bind a particular file -> ex -> -v C:\react_proj\service.txt:/app/service.txt (so only the service.txt file will be binded/connected)

## Concept

so basically all the files inside the container actually comes from C:\myapp\
and if u update the code in local repo program.js then that same code will be used by the container and immediately will run it. 
and vice versa if the container modifies a file (for example writing to names.txt),
the same change will appear in the host folder.

if we delete the container then it dosent delete the local repo project
container → deleted
C:\project → still exists
Because the files belong to the host, not the container.
The container only uses that folder/files from the host repo

The updates are Immediately Run in Conatiner.
and it also reflects new file created

## Key Idea

Bind mounts allow fast development and testing because:

- Code changes appear immediately
- No need to rebuild images
- Container can directly use host files

## Bind mount example with multiple containers 
we can make mulitple bind mount containers and all uses the local repo files and folder...  so if we update the local repo files ...or if we update any container repo files ... then it will be reflect in the local repo as well as all the container repo which was connected uisng blind mount.

host folder:
C:\shared_data

run container1:
docker run -v C:\shared_data:/app/data image

run container2:
docker run -v C:\shared_data:/app/data image

both containers access the same files.
so if container1 updates:
/app/data/file.txt
container2 can read it immediately.
because both are using the same host folder.

(so this same behaviour was also in the volumes where we were able to use the same data for differnt containers using the volume, but here its not just data but also other files and folders, and they are bind mounted/ connected with host repo also.)

## recommended syntax (modern
instead of -v, docker recommends: --mount

example:
docker run --mount type=bind,source=C:\project,target=/app image

meaning:
type   → bind mount
source → host folder
target → container folder
when bind mounts are used

----------------------------------------------------------------------------------------------------------------------------------
# .dockerignore

To ignore the files / folder not needed to put in docker image. 

ex- 
.git
.gitignore
dockerfile
etc...
So these folders / files will be ignored when making the image.  

----------------------------------------------------------------------------------------------------------------------------------
# communication 

## 1) Communicate with external APIs
Containers can access external APIs (internet services) directly, so no special configuration is needed. Just build the image and run the container and the application can call external APIs normally.

## 2) Communicate with a database on the host machine
Containers are isolated from the host system, so they cannot access services on the host using "localhost".
Use -> host = "host.docker.internal"

Example when writting SQL QUERIES :-
instead of this -> host = "localhost"
wirte -> host = "host.docker.internal"    // and then make image and run container

## 3) Communicate with a database running in another container
Here two containers need to communicate.

First start a MySQL container:
docker pull mysql
docker run -d \
--name mysqldb \
-e MYSQL_ROOT_PASSWORD=root \
-e MYSQL_DATABASE=userinfo \
mysql

You can find the container IP using:
docker inspect myslqdb  -> inspect this container, and u can see the Network:ip_address:172.32.234.3  -> so at this ip the sql container is running.

Then in the application code use that IP:
host = "172.x.x.x"
user = "root"
password = "root"
database = "userinfo"

Build and run the application container and it will connect to the MySQL container.

Note: Using container IP is not recommended because container IPs can change when containers restart.
Data: If the MySQL container is deleted, the database data will also be lost unless volumes are used.

## 4) Communicate with Database using same docker network (recommended)

In the previous method we used:
host = "172.32.234.3"
But this is not a good approach because:
container IP addresses change when container restarts
every restart may give a new IP
application may break
So Docker provides a better solution → custom networks.

concept:
If multiple containers are connected to the same docker network, then they can communicate using container names instead of IP address.
Example network -> Network : my-net
Containers connected to this network:
app_container  ←→  mysql_container
Now the application can access MySQL using:
host = "mysqldb"
because Docker automatically creates DNS for container names.

Step 1 — create a network
docker network create my-net
Check networks:
docker network ls

Step 2 — run mysql container in this network
docker run -d \
--name mysqldb \
--network my-net \
-e MYSQL_ROOT_PASSWORD=root \
-e MYSQL_DATABASE=userinfo \
mysql

Now MySQL container is running in my-net network.

Step 3 — write application code
Instead of:
host = "localhost"
or
host = "172.32.234.3"
write:
host = "mysqldb"
Because container name works as DNS hostname.
Other configs:
user = "root"
password = "root"
database = "userinfo"

Step 4 — build application image
docker build -t myapp .

Step 5 — run application container in same network
docker run -d \
--network my-net \
--name appcontainer \
myapp

Now both containers are inside  my-net so they can communicate.

-----MYsql container with Volumes -----
If MySQL container stores data inside container only:
delete mysql container → data lost
So normally we attach a volume:

docker run -v mysql-data:/var/lib/mysql ...

final architecture
------------my-net network-----------

   app_container  ←→  mysqldb_container
      (python)          (mysql)
-------------------------------------

----------------------------------------------------------------------------------------------------------------------------------
# Docker compose:

it is Configuration file to manage multiple containers running in same machine

When we build real applications we usually have multiple containers.
Example:
web application
database
redis cache
message queue

So problems start appearing: too many docker commands, Big Commands,  hard to manage, need to remember order, difficult to stop everything
Example:
docker network create my-net
docker run mysql container
docker run app container
docker run redis container
docker run nginx container
docker run -d -env MYSQL_ROOT_PASSWORD = "root" --env MYSQL_DATABASE="userinfo" --name mysqldb mysql

To solve this Docker Compose allows us to define multiple containers in one docker-compose.yaml / .yml file and run them together.

Example: 
version: "3"
services:
  mysqldb:
    image: mysql
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: userinfo
  app:
    build: .
    depends_on:
      - mysqldb

RUN:
docker-compose up -> Docker will Automatically: create network, create containers, start containers
docker-compose down -> for stopping containers. This stops and removes all containers created by compose.

so now Instead of writing many commands:
docker run ...
docker run ...
docker run ...
We write everything in one file -> docker-compose.yml
Then run everything using one command: -> docker compose up

Both containers automatically connect to the same Network. and no need to write it seperatly.
(in some cases when we run docker-compose up then it also builds image .. and then runs it. so then if we do docker-compose down then it removes the container but dosent delete that build image. and next time when we do docker-compose up then it uses that build image)

Advantages : 
easy multi-container setup
single command to run everything
automatic network creation
container communication by name
easy to start / stop project
good for development

## Commands:
docker-compose up -> start containers
docker-compose up -d -> start in background
docker-compose run service -> run that service container
docker-compose run -d service -> run that service container in background
docker-compose down -> stop containers
docker-compose ps -> view running services
docker-compose logs -> view logs
docker-compose up --build   -> rebuild containers

## ---------- docker-compose.yml Template -------

version: "3.9"   # docker compose file format version

services:

  service1_name/container_name:                       # name of first container/service (example: app)

    build: ./project_folder            # build image using Dockerfile inside this folder
    # OR
    # image: image_name:tag            # use prebuilt image from docker hub instead of building

    container_name: container_name     # optional custom container name (otherwise compose auto-generates)

    ports:
      - "host_port:container_port"     # example: "5000:5000"
                                       # host_port → port on your machine
                                       # container_port → port inside container

    volumes:
      - volume_name:container_path     # docker volume (persistent storage managed by docker)
      - ./local_folder:container_path  # bind mount (host folder connected to container folder)
                                       # example: ./project:/app

    environment:
      VAR1: value1
      VAR2: value2                     # environment variables passed to container

    working_dir: /container_folder     # working directory inside container (like WORKDIR in Dockerfile)

    command: run_command               # overrides CMD from Dockerfile
                                       # example: python app.py

    restart: always                    # restart policy
                                       # options → no | always | on-failure | unless-stopped

    depends_on:
      - service2_name                  # this service depends on service2
                                       # compose will start service2 before starting this service
                                       # if you run only this service using:
                                       # docker compose run service1_name
                                       # then docker compose will automatically start service2 first
                                       # NOTE: this only ensures start order, not that service2 is fully ready

    networks:
      - network_name                   # connect this container to custom network
                                       # if networks section is NOT written here,
                                       # docker compose automatically puts all services
                                       # - in the same default network so they can communicate
                                       # - using service names as hostnames

  service2_name:                       # second container/service (example: mysql / redis / backend)

    image: image_name:tag              # example: mysql:latest

    container_name: container_name

    environment:
      VAR1: value1
      VAR2: value2

    volumes:
      - volume_name:container_path

    networks:
      - network_name                   # same network so containers can communicate

---------- persistent storage (docker volumes) ----------
volumes:
  volume_name:                         # example: mysql_data
                                       # docker manages this storage location
                                       # used for database files or persistent app data

---------- custom docker network ----------
networks:
  network_name:                        # example: my-net
                                       # we define this when we want a specific custom network
                                       # otherwise docker compose automatically creates
                                       # a default network for the project

----------------------------------------------------------------------------------------------------------------------------------
# CONTAINER SCALING (CORE NOTES)

When an application gets more users, one container may not be enough to
handle all requests. Instead of increasing the power of a single container,
we usually run multiple containers of the same application.

This is called Horizontal Scaling.

Example architecture

Users
  |
Load Balancer
  |
  |----> Container A
  |----> Container B
  |----> Container C

All containers run the same image (same code, same environment). The load
balancer distributes incoming requests between them so that no single
container gets overloaded.

Example containers created from same image:
myapp:1.0  →  app1, app2, app3

RUN MULTIPLE CONTAINERS
docker run -d -p 3001:3000 --name app1 myapp:1.0
docker run -d -p 3002:3000 --name app2 myapp:1.0
docker run -d -p 3003:3000 --name app3 myapp:1.0

Common load balancers:
Nginx
HAProxy
Traefik
Cloud load balancers (AWS / GCP)

## SERVICE REPLICATION

Instead of manually running containers, orchestration systems can maintain
a fixed number of replicas.

Example

Service: backend
Replicas required: 3

Running containers:
backend-1
backend-2
backend-3

If one container crashes, the system automatically starts a new one so that
the number of replicas stays constant.
and if traffic increase then u can set -> Replicas required: 10 or more

## STATELESS vs STATEFUL

Stateless containers do not store data inside them.
Any container can handle any request, which makes scaling easy.
Examples:
web servers
APIs
microservices

Stateful containers store data.
If these containers are removed, data may be lost, so they require
persistent storage (volumes).
Examples:
databases
file storage

## CONTAINER ORCHESTRATION

When applications grow large, manually managing containers becomes difficult.

Problems:
- many containers to start/stop
- containers crashing
- scaling needed
- updates required
Orchestration systems automate this.

They manage:
- scaling containers
- restarting failed containers
- distributing containers across machines
- load balancing traffic
- rolling updates

Popular orchestrators:
Kubernetes
Docker Swarm
Nomad

## REAL WORLD SCALING FLOW 

1. Build application image
   myapp:1.0

2. Deploy containers
   app1
   app2
   app3

3. Put load balancer in front

4. If traffic increases
   run more containers

5. If a container crashes
   system starts another container automatically

----------------------------------------------------------------------------------------------------------------------------------
# summary : flow of dockerising a project

## 1) Make Project
   Write your application normally (Node / Python / FastAPI / Flask etc.)

## 2) Install Docker
   Install Docker Desktop and ensure Docker Engine is running.

## 3) Create Dockerfile
   This file tells Docker how to build the image.

   Basic template:

   FROM node:20-alpine              # base image (software environment -> node , pytohon , tensorflow)
   
   WORKDIR /myapp                   # working directory inside container

   COPY package*.json ./            # copy dependency files first
   
   RUN npm install                  # install dependencies (to run ternimal cmd to make image , pip install requirement.txt , etc ..)

   COPY . .                         # copy project code

   EXPOSE 3000                      # container port

   CMD ["npm","start"]              # command to run when container starts

## 4) Create .dockerignore
   Prevent unnecessary files from going into the image.

   Example:
   node_modules
   .git
   .env
   __pycache__

## 5) Update Project Code for Container Networking
   Some frameworks must listen on all interfaces.
   Example:
   FastAPI / Flask -> host = "0.0.0.0"
   Instead of -> localhost

## 6) Build Image

   docker build -t image_name:tag_name .

   Example: docker build -t myapp:v1 .

## 7) Run Container

   docker run -d -p host_port:container_port --name container_name image_name:tag_name

   Example:
   docker run -d -p 3000:3000 --name mycontainer myapp:v1

## 8) Debug 
(if container not working then update code, again make image, container )

   Useful commands:

   docker image ls -a               # view images
   docker image inspect image_name:tag # inspect a image
   docker ps -a                     # view containers
   docker logs container_name       # view container logs
   docker rmi image_id/name:tag   # delete image
   docker rm container_id/name    # delte container

## 9) Push Image to Docker Hub

   Create repository on Docker Hub.

   Run docker login in terminal -> docker login

   Rename the slected image to push as per the dockerhub repo name:
   docker tag local_image_name:tag username/repo_name:tag

   Example:
   docker tag myapp:v1 ujwal/myapp:v1

   Push that Renamed image:
   docker push username/repo_name:tag

   u can now delete the local image and container from ur machine:
   docker rm conainer
   docker rmi image 
   docker system prune -a

## 10) Run Image on Another Machine

   Install Docker. 
   copy the repo name of the image

   Open terminal or docker engine terminal

   Pull image from Docker Hub:
   docker pull username/repo_name:tag

   Example:
   docker pull ujwal/myapp:v1

   Run Container:
   docker run -d -p host_port:container_port --name container_name username/repo_name:tag

   Example:
   docker run -d -p 3000:3000 --name mycontainer ujwal/myapp:v1

----------------------------------------------------------------------------------------------------------------------------------
## --------------- Docker Explanation for Interview --------------

1) Docker Explain -> Docker is basically a containerization tool that lets us package an application along with all its dependencies and environment so it runs the same on any machine.
2) Example -> Sometimes an application works on a developer’s machine but fails on another system because of different dependency versions, OS setup, or missing packages. Docker solves this by packaging everything the app needs into a container.
3) Docker Architecture Full Explain -> In Docker we mainly work with three things: Dockerfile, Image, and Container. A Dockerfile defines how to build an Image and Image is Basically the packaged version of that Project with dependencies and etc all, and then using that Image we can Make and Run Multiple Containers.
4) Volumes -> Volumes are used to store data outside the container so that even if the container is deleted, the data remains safe.
5) Bind Mount -> Bind mount connects a folder from the host machine to the container so both can access the same files. This is mostly used during development.
6) Docker Network -> Containers are isolated, so Docker networks allow them to communicate with each other, usually using container names instead of IP addresses.
7) Docker Compose -> Docker Compose is used when an application needs multiple containers like a backend, database, and cache. Instead of running many commands, we define everything in a docker-compose.yml file and start everything together.
8) Docker Hub -> Docker Hub is basically a registry like GitHub but for Docker images. We can push our images there and others can pull them and run containers from them.
