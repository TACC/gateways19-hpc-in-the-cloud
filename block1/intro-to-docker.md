# Intro to Docker

## Lecture: What is Docker?

### What is a Container?

* Isolated Userland processes
* Virtualize: Network, I/O, CPU, and memory
* Rooted file system means all application assets contained within container; promotes:
  * Portability
  * Reproducibility
  * Ease of installation

<img src="../images/linux_containers_intro.png" alt="Linux Containers">

### Containers vs VMs
* Containers
  * OS process level isolation
  * Can run 1,000s of containers on a single machine
  * Leverages kernel features (requirements on kernel version)
  * Start up time ~100ms
  
* VMs
  * OS level isolation with virtualized hardware
  * Can run dozens of VMs on a single machine
  * Leverages hypervisors (requirements on physical hardware)
  * Start up time ~minutes

### The Docker Platform
Docker is a platform (among several) for building and executing containers.

* Images - Container “templates”. Essentially root filesystems with a little metadata (exposed ports, volumes, etc.)
* Container runtime - Create containers from images and run commands in them. 
* Docker Hub - Central, public repository of images.
* Additional Tooling: 
    * Additional client APIs - run commands in containers, get resources consumed, view logs
    * Docker Compose, Machine, Swarm - Tools for distributing containers across multiple hosts

### Exercise: Initial setup
Typically, accessing the docker daemon requires the user to be in the docker group. For the purposes of this introduction,
 we can simply do everything as the ubuntu user, which you are already logged in as. 


Make sure you can access the docker daemon; you can verify this by checking the version:
```
$ docker version
Client:
 Version:      1.13.1
 API version:  1.26
 Go version:   go1.6.2
 Git commit:   092cba3
 Built:        Thu Nov  2 20:40:23 2017
 OS/Arch:      linux/amd64

Server:
 Version:      1.13.1
 API version:  1.26 (minimum version 1.12)
 Go version:   go1.6.2
 Git commit:   092cba3
 Built:        Thu Nov  2 20:40:23 2017
 OS/Arch:      linux/amd64
 Experimental: false

```

Create a test directory to contain your docker work:
```
$ mkdir docker; cd docker
```

### Lecture: Docker Images and Tags, Docker Hub, and Pulling Images
A Docker image is a container template from which one or more containers can be run. It is a rooted filesystem that,
by definition, contains all of the file dependencies needed for whatever application(s) will be run within the
containers launched from it. The image also contains metadata describing options available to the operator running
containers from the image.

One of the great things about Docker is that a lot of software has already been packaged into Docker images. One source
of 100s of thousands of public images is the offcial docker hub: <https://hub.docker.com>

The docker hub contains images contributed by individual users and organizations as well as "official images". Explore
the offcial docker images here: <https://hub.docker.com/explore/>

For example, there is an official image for the Python programming language: <https://hub.docker.com/_/python/>

Docker supports the notion of image tags, similar to tags in a git repository. Tags identify a specific version of an
image.

The full name of an image on the Docker Hub is comprised of components separated by slashes. The components include a
"repository" (which could be owned by an individual or organization), the "name", and the "tag". For example, an image
with the full name

```
tapis/jupyter
```
would refer to the `jupyter` image within the "tapis" repository, if no tag is specified the "latest" tag is pulled. TACC maintains multiple repositories on the Docker Hub
including:
```
tacc
taccsciapps
tapis
abaco
```

### Exercise: Pulling and Running Images

Let's check that our docker installation is set up correctly by pulling the `tapis/jupyter` and image
and running a simple container from it:
```
# pull the image:
docker pull tapis/jupyter

# run a container:
docker run --rm -it -p 8888:8888 tapis/jupyter

```
We'll cover the `docker run` statement in more detail momentarily, but for now just know that it
should have started a single container from the `tapis/jupyter` image, which prints out a url to access the jupyter notebook from the browser.

###  Optional Exercises:
### Lecture: About Official Images
Official images such as the python official image are not owned by a repository, but all other images are.

To pull an image off Docker Hub use the `docker pull` command and provide the full image name:

```
$ docker pull python
Using default tag: latest
latest: Pulling from library/python
cc1a78bfd46b: Pull complete
. . .
```

As indicated in the output, if no tag is specified the "latest" tag is pulled. You can verify that the image is
available on your local machine using the `docker images` command:
```
$ docker images
REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
tapis/jupyter          latest              9dfe5a2c4b43        52 minutes ago      81.2 MB
python                 latest              a5b7afcfdcc8        3 hours ago         912 MB
```





