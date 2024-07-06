# AIME MLC Installation Guide


In this document we will describe how to install the AIME ML Containers (from now on, AIME MLC) on your server/workstation working with Ubuntu 22.04 LTS.


AIME MLC allows the management of Docker containers for Pytorch and Tensorflow for various versions and across platforms.


This is the NVIDIA / CUDA version of the installation, so a NVIDIA GPU must be available to use the capabilities of the AIME in MLCs and to check that all needed components are correctly installed.


A ROCM based installation guide for AIME MLC is currently in preparation.


For NVIDIA GPUs, the following components are required to be installed in this order:


1. [NVIDIA CUDA](#1-cuda-drivers-installationreinstallation): the installation of CUDA and compatible NVIDIA GPU drivers 

2. [Docker Engine](#2-docker-ce-engine-installation): the basic interface to create and manage containers 

3. [NVIDIA Container Toolkit](#3-nvidia-container-toolkit-installation): this toolkit allows you to use the GPU within a Docker container 

4. The [AIME ML Container solution](#4-aime-ml-containers-mlc-installationuninstallation) we offer to create and manage containers with the AI frameworks Pytorch and Tensorflow 



## Prerequisites of software and hardware:


- Ubuntu 22.04 LTS with sudo rights

- at least one NVIDIA GPU installed in your system


A clean system is not required, as we will show you how you can uninstall or remove previous installed NVIDIA drivers, CUDA and Docker versions to install AIME MLC successfully.


First of all update your Ubuntu using:

```

sudo apt-get update -y

sudo apt-get upgrade -y

sudo apt autoremove -y

```


## 1. CUDA Drivers installation/reinstallation:


The first step is to install the CUDA Toolkit including the NVIDIA driver. CUDA version 12.3 is the current recommended version as it is compatible with current Pytorch and Tensorflow versions. For how to install the latest CUDA version, please read below the Hints.


The installation of CUDA comes with a compatible NVIDIA GPU driver, so the NVIDIA driver should not be installed separately. In case you already have CUDA or a NVIDIA driver installed on your system, please check the topic "CUDA Clean deinstallation" how to get to the starting point.


[Here](https://developer.nvidia.com/cuda-12-3-0-download-archive) we have selected the options Linux / x86_64 / Ubuntu / 22.04 / deb(network) corresponding to our machine characteristics and chose the installer type: deb(network). The other options (operative system, architecture, distro and version) depend on your machine. After the selection of the above options appear the installation instructions:

```

 wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2204/x86_64/cuda-keyring_1.1-1_all.deb

 sudo dpkg -i cuda-keyring_1.1-1_all.deb

 sudo apt-get update -y

 sudo apt-get -y install cuda-12-3 # LINE MODIFIED!!

 sudo apt-get update -y

 sudo apt-get upgrade -y

 sudo apt autoremove -y

```



After installation, reboot the server/workstation:


```

sudo reboot

```

and check that the GPUs are visible using the command:


```

nvidia-smi

```


🔥 Hints 🔥 :


- We have modified above the line where the CUDA version is installed substituting:

    ```
    sudo apt-get -y install cuda-toolkit-12-3
    ```

    with


    ```
    sudo apt-get -y install cuda-12-3
    ```


    With this modification we assure that not only the cuda-toolkit is installed but the complete cuda-12-3 setup including the NVIDIA driver.


- In case that you prefer the installation of the last CUDA version, use this [link](https://developer.nvidia.com/cuda-downloads).


- It is highly recommended to install CUDA by giving a defined version number:

    ```
    sudo apt-get -y install cuda-12-3
    ```


    instead of


    ```
    sudo apt-get -y install cuda
    ```


### Determine installed CUDA version


In case you are not sure if or which CUDA version is installed on your system, try following command:


```

sudo apt list --installed | grep cuda-1

```


which will print something like this, in case a cuda version is already installed:


```

WARNING: apt does not have a stable CLI interface. Use with caution in scripts.


cuda-12-1/unknown,now 12.1.1-1 amd64 [installed]

```


which states that, in this case, CUDA 12.1 is installed.



### Reinstalling or Upgrading CUDA


Imagine that you would like to downgrade or upgrade your current CUDA version (for example from CUDA 12.5 to CUDA 12.3). As a new installation of the cuda-keyring_1.1-1_all is not needed anymore (just installed), first of all you must remove the current CUDA version before you install the new one using:


```
 sudo apt-get remove cuda-12-5

 sudo apt-get autoremove -y

 sudo apt-get update -y

 sudo apt-get upgrade -y

 sudo reboot
```


 Now you can install CUDA 12.3:


```
 sudo apt-get install cuda-12-3

 sudo apt-get update -y

 sudo apt-get upgrade -y

 sudo reboot

```




### CUDA Clean deinstallation: in case that you wish to uninstall CUDA including the GPU driver completely


We would recommend to first try to uninstall CUDA by version, if this fails use the method to uninstall CUDA by uninstalling the NVIDIA driver.


#### Uninstall CUDA by version


```

 sudo apt-get remove cuda-XX-X # insert here the version to be removed

 sudo apt-get purge cuda-keyring -y

 sudo apt-get autoremove -y

 sudo apt-get update -y

 sudo apt-get upgrade -y

```


If there is no error message this uninstallation method was successful and is sufficient.


#### Uninstall NVIDIA driver (including CUDA)


In some cases CUDA is only partly installed or there were problems installing CUDA because an incompatible NVIDIA driver is already installed on the system. So to start with a clean starting point for installation the NVIDIA driver and the remaining parts of the installed CUDA toolkit should be removed completely with following commands:


```

sudo apt-get --purge remove "*cublas*" "cuda*"

sudo apt-get --purge remove "*nvidia*"

sudo apt-get autoremove -y

```

Take into account that after the CUDA clean deinstallation (NVIDIA Driver including CUDA) you have to install again the NVIDIA Container Toolkit, in case that you have installed it before.


## 2. Docker CE (Engine) installation


The next step is the installation of the Docker Engine and to enable it for use as non root user.

1. Update the apt package and install needed packages:

        sudo apt-get update -y

        sudo apt-get install ca-certificates curl gnupg lsb-release -y


2. Add Docker’s official GPG key:

        sudo install -m 0755 -d /etc/apt/keyrings

        sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc

        sudo chmod a+r /etc/apt/keyrings/docker.asc

3. Add the repository to apt sources:

        echo \
        "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
        $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
        sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

4. Now is time to Install Docker Engine and related packages:

        sudo apt-get update -y
        sudo apt-get install docker-ce docker-ce-cli containerd.io -y

5. Start the Docker service on the system:

        sudo systemctl --now enable docker

6. Verify the installed version using:

        docker --version

7. The next, we will use [Docker as non-root user](https://docs.docker.com/install/linux/linux-postinstall/#manage-docker-as-a-non-root-user). For that, we will add the current user name to the docker group:

        sudo adduser $USER docker
        newgrp docker


8. Verify that you can run Docker commands without sudo :

        docker run hello-world


🔥 Hints🔥:


- don't install the packages docker-buildx-plugin docker-compose-plugin, as in the [instructions](https://docs.docker.com/engine/install/ubuntu/) is explained.


### How to uninstall Docker


In case a non working or old version of Docker is already installed on your system, follow the recommended steps from [Docker](https://docs.docker.com/install/linux/docker-ce/ubuntu/):


- Uninstall the Docker Engine, CLI, containerd, and Docker Compose packages:


```
    sudo apt-get purge docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin docker-ce-rootless-extras -y

    sudo apt autoremove -y

```


- Images, containers, volumes, or custom configuration files on your host aren't automatically removed. To delete all images, containers, and volumes ( 🚨 backup needed?? 🚨 ):


```
    sudo rm -rf /var/lib/docker

    sudo rm -rf /var/lib/containerd
```


- A good idea is to uninstall any conflicting packages, before you can install Docker Engine again:


```
    for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done
```


- You have to delete any edited configuration files manually.


- To verify that Docker is not anymore installed, you can use one of the following commands:


```
    docker version

    systemctl status docker

    sudo apt list --installed | grep docker
```


## 3. NVIDIA Container Toolkit installation


The next step is the installation of the NVIDIA Docker, the so-called [NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html#docker), to be able to use the GPUs within Docker.


1. First, we configure the production repository without experimental features:


```
    curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg \
  && curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
    sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
    sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list

```

2. Now we update the package list from the repository and install the NVIDIA Container Toolkit packages:


    `sudo apt-get update -y`

    `sudo apt-get install nvidia-container-toolkit -y`

3. After the installation of the Container Toolkit, Docker must be configured using nvidia-ctk command:


    `sudo nvidia-ctk runtime configure --runtime=docker`


    The nvidia-ctk command modifies the /etc/docker/daemon.json file on the host. The file is updated so that Docker can use the NVIDIA Container Runtime.

5. Now you can restart the Docker daemon to complete the installation after setting the default runtime:


    `sudo systemctl restart docker`

6. To be sure that all needed packages are installed, use again:

    `sudo apt-get update -y`

    `sudo apt-get upgrade -y`


### Uninstall NVIDIA Container Toolkit


By following these steps, you can completely uninstall the NVIDIA Container Toolkit from your system.


- First, you need to remove the nvidia-container-toolkit and nvidia-container-runtime package using your package manager:


```
    sudo apt-get remove --purge nvidia-container-toolkit -y
    
    sudo apt-get remove --purge nvidia-container-runtime

    sudo apt-get autoremove -y
```

- After removing the packages, you may want to clean up any residual configuration files:

```
    sudo rm -rf /etc/docker/daemon.json /etc/systemd/system/docker.service.d/nvidia-container-runtime* /etc/nvidia-container-runtime
```

- Now, you should restart the Docker daemon to ensure it no longer NVIDIA runtime is used:


```
    sudo systemctl restart docker
```

- Finally, verify that the NVIDIA Container Toolkit and its runtime have been successfully removed:


```
    dpkg -l | grep nvidia
```

- And ensure that the NVIDIA runtime is not present in Docker’s configuration:


```
    docker info | grep -i nvidia
```


Take into account that after the deinstallation of the NVIDIA Container Toolkit, AIME MLCs can not be opened anymore ( 🚨 backup needed 🚨 ).



## 4. AIME ML Containers (MLC) installation/uninstallation


Before we start the actual installation of AIME MLC, which is the last step of this installation guide, we have to create the workspace folder which is the default mounting point for your source and project files into the ML container:


```

mkdir /home/$USER/workspace

```


Now we clone the aime-ml-containers from AIME's [Github](https://github.com/aime-team/aime-ml-containers) web site, selecting the most appropriate branch for your installed GPU, on the location you prefer. The recommended location would be somewhere where you don't need sudo rights, e.g. within the home directory:


```

git clone --branch stable-cuda-ada https://github.com/aime-team/aime-ml-containers.git /home/$USER/aime-ml-containers

```


AIME provides different branches for the AIME MLCs depending on the model of the GPU, NVIDIA or AMD, and the generation:


- stable-cuda-ada: used for the current NVIDIA GPU generation "Ada Lovelace and Hopper" like RTX 4090, RTX 5000/ 6000 ada, H100 it is currently backward compatible with Ampere based GPUs, but does not provide earlier version only available for the Ampere generation


- stable-cuda-ampere: for the NVIDIA Ampere generation, like RTX 3090, RTX A5000/A6000, A100


- stable-cuda: for the NVIDIA Turing GPU generation, like RTX 2080TI, RTX Titan, Quadro RTX


- stable-rocm6: for AMD ROCM 6.0 supported GPUs


- stable-rocm5: for AMD ROCM 5.0 supported GPUs


The next step is to add in /home/user/.bashrc the path where aime-ml-containers was cloned to, in the above example /home/$USER/aime-ml-containers, adding the following at the end of .bashrc:


```

if [ -d "/home/$USER/aime-ml-containers" ] ; then


PATH="$PATH:/home/$USER/aime-ml-containers"


fi

```

and source it using:

`source .bashrc`


🔥 Hints 🔥 :


- Check that the AIME MLCs work correctly updating to the latest version using:


    ```
    mlc-update-sys
    ```

    Your should see something similar to:

    `This will update the ML container system to the latest version.`

    `Check for available updates? (Y/N) y`

    `[sudo] password for user-mlc:`

    `ML container system is up to date.`


    The AIME MLCs were not correctly installed if you see:

```
    mlc-update-sys: Command not found
```




Now you can take advantage of all functionalities of AIME MLCs, starting with the creation of a MLC using mlc-create and choosing one of your favorite deep learning frameworks, Pytorch or Tensorflow.



More information about how to work with AIME MLCs, [here](https://www.aime.info/blog/en/deep-learning-framework-container-management/).




### Uninstall AIME MLC


In case you want to uninstall AIME MLC, juste remove the path from the .bashrc file and delete the folder where you cloned the aime-ml-containers files, in this example, /home/$USER/aime-ml-containers.


Before that, we would recommend to remove all previous created MLCs by using mlc-remove container_name.


After that, to reclaim the disk space allocated by Docker and release the cached downloaded docker container images with:


```

'docker image prune --all'

```
