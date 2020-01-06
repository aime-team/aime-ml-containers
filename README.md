# AIME ML Containers

AIME machine learning container management system. 

Easily install, run and manage docker containers for the most common deep learning frameworks. 

## Features

* Setup and run a specific version of Tensorflow, Pytorch or Mxnet with one simple command
* Run different versions of machine learning frameworks and required libraries in parallel
* manages required libraries (CUDA, CUDNN, CUBLAS, etc.) in containers, without compromising the host installation
* Clear separation of user code and framework installation, test your code with a different framework version in minutes
* multi session: open and run many shell session on a single container simultaneously
* multi user: separate container space for each user
* multi GPU: allocate GPUs per user, container or session 
* Runs with the same performance as a bare metal installation
* Repository of all major deep learning framework versions as containers

## Usage

### Create a machine learning container

**mlc-create container_name framework version [-w=workspace]**

Create a new machine learning container

Available Frameworks:

Tensorflow, Mxnet, Pytorch

Available Versions:

Tensorflow: 2.0.0, 1.15.0, 1.15, 1.14.0, 1.13.2

Mxnet: 1.5.0, 1.4.0, 1.3.0

Pytorch: 1.3.0, 1.2.0, 1.1.0 

Example to create a container with the name 'my-container' as Tensorflow 1.15.0 with mounted user home directory as workspace use:

```
> mlc-create my-container Tensorflow 1.15.0 -w=/home/admin
```


### Open a machine learning container

**mlc-open container_name** 

To open the created machine learning container "my-container"

```
> mlc-open my-container
```

Will output:

```
[my-container] starting container
[my-container] opening shell to container

________                               _______________
___  __/__________________________________  ____/__  /________      __
__  /  _  _ \_  __ \_  ___/  __ \_  ___/_  /_   __  /_  __ \_ | /| / /
_  /   /  __/  / / /(__  )/ /_/ /  /   _  __/   _  / / /_/ /_ |/ |/ /
/_/    \___//_/ /_//____/ \____//_/    /_/      /_/  \____/____/|__/


You are running this container as user with ID 1000 and group 1000,
which should map to the ID and group for your user on the Docker host. Great!

[my-container] admin@aime01:/workspace$
```

The container is run with the access rights of the user. To use privileged rights like for installing packages with 'apt' within the container use 'sudo'. The default is that no password is needed for sudo, to change this behaviour set a password with 'passwd'.

Multiple instances of a container can be opened with mlc-open. Each instance runs in its own process.

To exit an opened shell to the container type 'exit' on the command line. The last exited shell will automatically stop the container.


### List avilable machine learning containers

**mlc-list** will list all available containers for the current user


```
> mlc-list
```

will output for example:

```
Available ml-containers are:

CONTAINER           FRAMEWORK                  STATUS
[torch-vid2vid]     Pytorch-1.2.0              Up 2 days
[tf1.15.0]          Tensorflow-1.15.0          Up 8 minutes
[mx-container]      Mxnet-1.5.0                Exited (137) 1 day ago
[tf1-nvidia]        Tensorflow-1.14.0_nvidia   Exited (137) 1 week ago
[tf1.13.2]          Tensorflow-1.13.2          Exited (137) 2 weeks ago
[torch1.3]          Pytorch-1.3.0              Exited (137) 3 weeks ago
[tf2-gpt2]          Tensorflow-2.0.0           Exited (137) 7 hours ago
```

### List active machine learning containers

**mlc-stats** show all current running ml containers and there CPU and memory usage

```
> mlc-stats

Running ml-containers are:

CONTAINER           CPU %               MEM USAGE / LIMIT
[torch-vid2vid]     4.93%               8.516GiB / 63.36GiB
[tf1.15.0]          7.26%               9.242GiB / 63.36GiB
```

### Start machine learning container

**mlc-start container_name** to explicitly start a container

mlc-stop will power up the container. mlc-start is a way to start the container to run installed background process like an installed web server on the container without the need to open an interactive shell to it. 

For opening a shell to the container just use 'mlc-open', which will automatically also start the container if necessary.


### Stop machine learning containers

**ml-stop container_name [-Y]** to explicitly stop a container. 

mlc-stop on a container is comparable to a shutdown of a computer, all activate processes and open shells to the container will be terminated.

To force a stop on a container use:

```
mlc-stop my-container -Y
```

### Remove/Delete a machine learning container

**mlc-remove container_name** to remove the container.

Warning: the container will be unrecoverable deleted only data stored in the /workspace directory will be kept. Only use to clean up containers which are not needed anymore. 

```
mlc-remove my-container
```
