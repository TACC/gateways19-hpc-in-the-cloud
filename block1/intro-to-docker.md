# Intro to Docker

## What is Docker?

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
Typically, accessing the docker daemon requires the user to be in the docker group.

Make sure you can access the docker daemon; you can verify this by checking the version:
```
$ docker version
Client:

 Cloud integration: 1.0.17
 Version:           20.10.7
 API version:       1.41
 Go version:        go1.16.4
 Git commit:        f0df350
 Built:             Wed Jun  2 11:56:22 2021
 OS/Arch:           darwin/amd64
 Context:           default
 Experimental:      true


Server: Docker Engine - Community
 Engine:
  Version:          20.10.7
  API version:      1.41 (minimum version 1.12)
  Go version:       go1.13.15
  Git commit:       b0f5bc3
  Built:            Wed Jun  2 11:54:58 2021
  OS/Arch:          linux/amd64
  Experimental:     true
 containerd:
  Version:          1.4.6
  GitCommit:        d71fcd7d8303cbf684402823e425e9dd2e99285d
 runc:
  Version:          1.0.0-rc95
  GitCommit:        b9ee9c6314599f1b4a7f497e1f1f856fe433d3b7
 docker-init:
  Version:          0.19.0
  GitCommit:        de40ad0

```

Create a test directory to contain your docker work:
```
$ mkdir docker; cd docker
```

### Docker Images, Tags, and Docker Hub
A Docker image is a container template from which one or more containers can be run. It is a rooted filesystem that,
by definition, contains all the file dependencies needed for whatever application(s) will be run within the
containers launched from it. The image also contains metadata describing options available to the operator running
containers from the image.

One of the great things about Docker is that a lot of software has already been packaged into Docker images. One source
of 100s of thousands of public images is the official docker hub: <https://hub.docker.com>

The docker hub contains images contributed by individual users and organizations as well as "official images". Explore
the official docker images here: <https://hub.docker.com/explore/>

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

Let's check that our docker installation is set up correctly by running the hello-world
```
# run hello world
docker run hello-world

You should see below message:
Hello from Docker!
This message shows that your installation appears to be working correctly.

```

Next, let pull the `tapis/jupyter` image and running a simple container from it:
```
# pull the image:
docker pull tapis/jupyter

# run a container:
docker run --rm -it -p 8888:8888 tapis/jupyter

```
It should have started a single container from the `tapis/jupyter` image, which prints out a url to access the jupyter notebook from the browser.
You can copy paste this url in your browser and access the jupyter notebook. We will learn about Jupyter shortly in this tutorial.

###  Optional Exercises:
### About Official Images
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
tapis/jupyter          latest              f3cb30161a63        4 days ago          1.25GB
python                 latest              a5b7afcfdcc8        3 hours ago         912 MB
```





