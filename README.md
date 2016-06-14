## Docker Image for Machine Learning & Data Science Toolkit
This includes deep learning frameworks with CPU and GPU support (CUDA and cuDNN included). The CPU version should work on Linux, Windows and OS X. The GPU version will, however, only work on Linux machines. See [OS support](#what-operating-systems-are-supported) for details

If you are not familiar with Docker, but would still like an all-in-one solution, start here: [What is Docker?](#what-is-docker). If you know what Docker is, but are wondering why we need one for deep learning, [see this](#why-do-i-need-a-docker)

## Included Items
When building with the Dockerimage file you get the folloiwing:
* Ubuntu 14.04
* [CUDA 7.5](https://developer.nvidia.com/cuda-toolkit) (GPU version only)
* [cuDNN v4](https://developer.nvidia.com/cudnn) (GPU version only)
* [Tensorflow](https://www.tensorflow.org/)
* [Caffe](http://caffe.berkeleyvision.org/)
* [Theano](http://deeplearning.net/software/theano/)
* [Keras](http://keras.io/)
* [Lasagne](http://lasagne.readthedocs.io/en/latest/)
* [Torch](http://torch.ch/) (includes nn, cutorch, cunn and cuDNN bindings)
* [iPython/Jupyter Notebook](http://jupyter.org/) (including iTorch kernel)
* [R-Base](https://www.r-project.org/) (including the bindings and packages required)
* [Rstudio](https://www.rstudio.com/products/rstudio/download-server/)
* [Numpy](http://www.numpy.org/), [SciPy](https://www.scipy.org/), [Pandas](http://pandas.pydata.org/), [Scikit Learn](http://scikit-learn.org/), [Matplotlib](http://matplotlib.org/)
* A few common libraries used for deep learning

## Setup
### Prerequisites
1. Install Docker following the installation guide for your platform: [https://docs.docker.com/engine/installation/](https://docs.docker.com/engine/installation/)

You can also use the following Docker installation script if you're using Ubuntu

```
wget -qO- https://get.docker.com/ | sh
sudo usermod -aG docker $(whoami)
```

2. **GPU Version Only**: Install Nvidia drivers on your machine either from [Nvidia](http://www.nvidia.com/Download/index.aspx?lang=en-us) directly or follow the instructions [here](https://github.com/saiprashanths/dl-setup#nvidia-drivers). Note that you _don't_ have to install CUDA or cuDNN. These are included in the Docker container.

3. **GPU Version Only**: Install nvidia-docker: [https://github.com/NVIDIA/nvidia-docker](https://github.com/NVIDIA/nvidia-docker), following the instructions [here](https://github.com/NVIDIA/nvidia-docker/wiki/Installation). This will install a replacement for the docker CLI. It takes care of setting up the Nvidia host driver environment inside the Docker containers and a few other things.

### Building the Docker containers
#### Build the Docker image locally
Now you can build the images locally. Note that this will take an hour or two depending on your machine since it compiles a few libraries from scratch.

```bash
git clone https://github.com/OverscoreDev/deep-docker.git
cd deep-docker
```

**CPU Version**
Note the name/tags that are used and you can use your own
```bash
docker build -t deepdocker:cpu -f Dockerfile.cpu .
```

**GPU Version**
Note the name/tags that are used and you can use your own
```bash
docker build -t deepdocker:gpu -f Dockerfile.gpu .
```
This will build a Docker image named `deepdocker` and tagged either `cpu` or `gpu` depending on the tag your specify. Also note that the appropriate `Dockerfile.<architecture>` has to be used.

## Running the Docker image as a Container
Once we've built the image, we have all the frameworks we need installed in it. We can now spin up one or more containers using this image.

**CPU Version**
```bash
docker run -p 8888:8888 -p 6006:6006 -p 8787:8787 -p 2222:22 -v /sharedfolder:/root/sharedfolder -it deepdocker:cpu bash
```

**GPU Version**
```bash
nvidia-docker run -it -p 8888:8888 -p 6006:6006 -p 8787:8787 -v /sharedfolder:/root/sharedfolder floydhub/dl-docker:gpu bash
```
Note the use of `nvidia-docker` rather than just `docker`

| Parameter      | Explanation |
|----------------|-------------|
|`-it`             | This creates an interactive terminal you can use to iteract with your container |
|`-p 8888:8888 -p 6006:6006 -p 8787:8787 -p 2222:22`    | This exposes the ports inside the container so they can be accessed from the host. The format is `-p <host-port>:<container-port>`. The default iPython Notebook runs on port 8888 and Tensorboard on 6006 and SSH on 22 and Rstudio 8787 |
|`-v /sharedfolder:/root/sharedfolder/` | This shares the folder `/sharedfolder` on your host machine to `/root/sharedfolder/` inside your container. Any data written to this folder by the container will be persistent. You can modify this to anything of the format `-v /local/shared/folder:/shared/folder/in/container/`. See [Docker container persistence](#docker-container-persistence)
|`deepdocker:cpu`   | This the image that you want to run. The format is `image:tag`. In our case, we use the image `dl-docker` and tag `gpu` or `cpu` to spin up the appropriate image |
|`bash`       | This provides the default command when the container is started. Even if this was not provided, bash is the default command and just starts a Bash session. You can modify this to be whatever you'd like to be executed when your container starts. For example, you can execute `docker run -it -p 8888:8888 -p 6006:6006 deepdocker:cpu jupyter notebook`. This will execute the command `jupyter notebook` and starts your Jupyter Notebook for you when the container starts

## Starting up daemons
I've made it a bit easier to start up the daemons required with a single bash script. You can run this through
```bash
sh run_all.sh
```
See the [To do section](#To Do) for the supervisor.conf implementation. This is still underway

### Data Sharing
See [Docker container persistence](#docker-container-persistence).
Consider this: You have a script that you've written on your host machine. You want to run this in the container and get the output data (say, a trained model) back into your host. The way to do this is using a [Shared Volumne](#docker-container-persistence). By passing in the `-v /sharedfolder/:/root/sharedfolder` to the CLI, we are sharing the folder between the host and the container, with persistence. You could copy your script into `/sharedfolder` folder on the host, execute your script from inside the container (located at `/root/sharedfolder`) and write the results data back to the same folder. This data will be accessible even after you kill the container.

## What is Docker?
[Docker](https://www.docker.com/what-docker) itself has a great answer to this question.

Docker is based on the idea that one can package code along with its dependencies into a self-contained unit. In this case, we start with a base Ubuntu 14.04 image, a bare minimum OS. When we build our initial Docker image using `docker build`, we install all the deep learning frameworks and its dependencies on the base, as defined by the `Dockerfile`. This gives us an image which has all the packages we need installed in it. We can now spin up as many instances of this image as we like, using the `docker run` command. Each instance is called a _container_. Each of these containers can be thought of as a fully functional and isolated OS with all the deep learning libraries installed in it.

## Why do I need a Docker?
Installing all the deep learning frameworks to coexist and function correctly is an exercise in dependency hell. Unfortunately, given the current state of DL development and research, it is almost impossible to rely on just one framework. This Docker is intended to provide a solution for this use case.

### Do I really need an all-in-one container?
No. The provided all-in-one solution is useful if you have dependencies on multiple frameworks (say, load a pre-trained Caffe model, finetune it, convert it to Tensorflow and continue developing there) or if you just want to play around with the various frameworks.

The Docker philosophy is to build a container for each logical task/framework. If we followed this, we should have one container for each of the deep learning frameworks. This minimizes clashes between frameworks and is easier to maintain as things evolve. In fact, if you only intend to use one of the frameworks, or at least use only one framework at a time, follow this approach. You can find Dockerfiles for individual frameworks here:
* [Tensorflow Docker](https://github.com/tensorflow/tensorflow/tree/master/tensorflow/tools/docker)
* [Caffe Docker](https://github.com/BVLC/caffe/tree/master/docker)
* [Theano Docker](https://github.com/Kaixhin/dockerfiles/tree/master/cuda-theano)
* [Keras Docker](https://github.com/Kaixhin/dockerfiles/tree/master/cuda-keras/cuda_v7.5)
* [Lasagne Docker](https://github.com/Kaixhin/dockerfiles/tree/master/cuda-lasagne/cuda_v7.5)
* [Torch Docker](https://github.com/Kaixhin/dockerfiles/tree/master/cuda-torch)

## To do

There are multiple items that we still need to look at such as

* Docker-compose - Multi-container support in the event that you need to embed certain parts of this docker into an application

* Supervisord configuration - There is an existing supervisord.conf but this needs to be updated to include the scripts that start all the services and monitor / auto restart

* Clean up all log directories and set the pathing properly here

* Installation of Shiny Server -  Is this required?

## FAQs
### Performance
Running the DL frameworks as Docker containers should have no performance impact during runtime. Spinning up a Docker container itself is very fast and should take only a couple of seconds or less

### Docker container persistence
Keep in mind that the changes made inside Docker container are not persistent. Lets say you spun up a Docker container, added and deleted a few files and then kill the container. The next time you spin up a container using the same image, all your previous changes will be lost and you will be presented with a fresh instance of the image. This is great, because if you mess up your container, you can always kill it and start afresh again. It's bad if you don't know/understand this and forget to save your work before killing the container. There are a couple of ways to work around this:

1. **Commit**: If you make changes to the image itself (say, install a few new libraries), you can commit the changes and settings into a new image. Note that this will create a new image, which will take a few GBs space on your disk. In your next session, you can create a container from this new image. For details on commit, see [Docker's documentaion](https://docs.docker.com/engine/reference/commandline/commit/).

2. **Shared volume**: If you don't make changes to the image itself, but only create data (say, train a new Caffe model), then commiting the image each time is an overkill. In this case, it is easier to persist the data changes to a folder on your host OS using shared volumes. Simple put, the way this works is you share a folder from your host into the container. Any changes made to the contents of this folder from inside the container will persist, even after the container is killed. For more details, see Docker's docs on [Managing data in containers](https://docs.docker.com/engine/userguide/containers/dockervolumes/)

### How do I update/install new libraries?
You can do one of:

1. Modify the `Dockerfile` directly to install new or update your existing libraries. You will need to do a `docker build` after you do this. If you just want to update to a newer version of the DL framework(s), you can pass them as CLI parameter using the --build-arg tag ([see](-v /sharedfolder:/root/sharedfolder) for details). The framework versions are defined at the top of the `Dockerfile`. For example, `docker build -t dl-docker:cpu -f Dockerfile.cpu --build-arg TENSORFLOW_VERSION=0.9.0rc0 .`

2. You can log in to a container and install the frameworks interactively using the terminal. After you've made sure everything looks good, you can commit the new contains and store it as an image

### What operating systems are supported?
Docker is supported on all the OSes mentioned here: [Install Docker Engine](https://docs.docker.com/engine/installation/) (i.e. different flavors of Linux, Windows and OS X). The CPU version (Dockerfile.cpu) will run on all the above operating systems. However, the GPU version (Dockerfile.gpu) will only run on Linux OS. This is because Docker runs inside a virtual machine on Windows and OS X. Virtual machines don't have direct access to the GPU on the host. Unless PCI passthrough is implemented for these hosts, GPU support isn't available on non-Linux OSes at the moment.

## Credits
* Credit goes out to Sai  Soundararaj for the initial image and readme 
* Rocker R  Dockerfile images (https://hub.docker.com/r/rocker/rstudio/~/dockerfile/)
