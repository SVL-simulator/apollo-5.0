# LG Silicon Valley Lab Apollo 5.0 Fork
This repository is a fork of [Apollo](https://github.com/ApolloAuto/apollo) maintained by the LG Electronics Silicon Valley Lab which has modified and configured to facilitate use with [LG's Automotive Simulator](https://github.com/lgsvl/simulator).

**The software and source code in this repository are intended only for use with LG Automotive Simulator and *should not* be used in a real vehicle.**

## Table of Contents

1. [Getting Started](#getting-started)
2. [Prerequisites](#prerequisites)
3. [Setup](#setup)
    - [Docker](#docker)
    - [Cloning the Repository](#cloning-the-repository)
    - [Building Apollo and Bridge](#building-apollo-and-bridge)
4. [Launching Apollo Alongside the Simulator](#launching-apollo-alongside-the-simulator)

## Getting Started
The guide outlines the steps required to setup Apollo for use with the LG Automotive Simulator. If you have not already set up the simulator, please do so first by following the instructions [here](https://github.com/lgsvl/simulator).

## Prerequisites
* Linux operating system (preferably Ubuntu 14.04 or later)
* Nvidia graphics card (required for Perception)
    - Nvidia proprietary driver must be installed
    - The current version of Apollo does not support Volta and Turing architectures (this includes Titan V and RTX 2080 GPUs).



## Setup

### Docker
Apollo is designed to run out of docker containers. The image will mount this repository as a volume so the image will not need to be rebuilt each time a modification is made.

#### Installing Docker CE
To install Docker CE please refer to the [official documentation](https://docs.docker.com/install/linux/docker-ce/ubuntu/).
We also suggest following through with the [post installation steps](https://docs.docker.com/install/linux/linux-postinstall/).

#### Installing Nvidia Docker
Before installing nvidia-docker make sure that you have an appropriate Nvidia driver installed.
To test if nvidia drivers are properly installed enter `nvidia-smi` in a terminal. If the drivers are installed properly an output similar to the following should appear.
```
    +-----------------------------------------------------------------------------+
    | NVIDIA-SMI 390.87                 Driver Version: 390.87                    |
    |-------------------------------+----------------------+----------------------+
    | GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
    | Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
    |===============================+======================+======================|
    |   0  GeForce GTX 108...  Off  | 00000000:65:00.0  On |                  N/A |
    |  0%   59C    P5    22W / 250W |   1490MiB / 11175MiB |      4%      Default |
    +-------------------------------+----------------------+----------------------+
                                                                                
    +-----------------------------------------------------------------------------+
    | Processes:                                                       GPU Memory |
    |  GPU       PID   Type   Process name                             Usage      |
    |=============================================================================|
    |    0      1187      G   /usr/lib/xorg/Xorg                           863MiB |
    |    0      3816      G   /usr/bin/gnome-shell                         305MiB |
    |    0      4161      G   ...-token=7171B24E50C2F2C595566F55F1E4D257    68MiB |
    |    0      4480      G   ...quest-channel-token=3330599186510203656   147MiB |
    |    0     17936      G   ...-token=5299D28BAAD9F3087B25687A764851BB   103MiB |
    +-----------------------------------------------------------------------------+
```

The installation steps for nvidia-docker are available at the [official repo](https://github.com/NVIDIA/nvidia-docker). 

#### Pulling LGSVL Docker image
LGSVL maintains a docker image to be used alongside this repository. The docker image is available [here](https://hub.docker.com/r/lgsvl/apollo-5.0/).

To pull the image use the following command:

    docker pull lgsvl/apollo-5.0

### Cloning the Repository
This repository includes a couple of submodules for HD Maps and lgsvl msgs. To make sure that the submodules are also cloned use the following command:

    git clone --recurse-submodules https://github.com/lgsvl/apollo-5.0.git


### Building Apollo and bridge
Now everything should be in place to build Apollo. Apollo must be built from the container. To launch the container navigate to the directory where the repository was cloned and enter:

    ./docker/scripts/dev_start.sh

This should launch the container and mount a few volumes. It could take a few minutes to pull the latest volumes on the first run.

To get into the container:

    ./docker/scripts/dev_into.sh

Build Apollo:

    ./apollo.sh build_gpu


### Building and Publishing a Runtime Docker Image

After you have tested your changes to Apollo 5.0 locally, you can create a much smaller docker image, which cannot be used to re-build modified Apollo, but can be shared with other people using it to run Apollo without the need to build it locally. You can also publish it to a publicly accessible docker registry (e.g., [Docker Hub](https://hub.docker.com/)).

To build the docker image, run:

    ./docker/build/runtime.x86_64.sh

If the development container is not running, you will see this message:

    Error: No such container: apollo_dev_USER
    ERROR: apollo_dev_USER isn't running or doesn't have /apollo/bazel directory
           make sure it's running (you can use docker/scripts/dev_start.sh)
           and build Apollo there or add "rebuild" parameter to this script
           and it will be started and built automatically

Upon successful completion, you will see:

    ...
    Docker image with prebuilt files was built and tagged as lgsvl/apollo-5.0-runtime:latest, you can start it with:
      docker/scripts/runtime_start.sh
    and switch into it with:
      docker/scripts/runtime_into.sh

Confirm operation of the image by starting it:

    ./docker/scripts/runtime_start.sh

entering into it:

    ./docker/scripts/runtime_into.sh

and running the appropriate commands to verify its functionality.

Finally, tag the image and push it to the publicly accessible Docker registry:

    docker image tag lgsvl/apollo-5.0-runtime:latest REGISTRY/IMAGE:TAG
    docker image push REGISTRY/IMAGE:TAG

To use such an image from Docker registry (without git checkout of Apollo 5.0 repository):

    docker image pull REGISTRY/IMAGE:TAG
    docker image tag REGISTRY/IMAGE:TAG lgsvl/apollo-5.0-runtime

    mkdir -p docker/scripts scripts modules/map/data

    wget https://raw.githubusercontent.com/lgsvl/apollo-5.0/simulator/docker/scripts/runtime_start.sh -O docker/scripts/runtime_start.sh
    wget https://raw.githubusercontent.com/lgsvl/apollo-5.0/simulator/docker/scripts/runtime_into.sh -O docker/scripts/runtime_into.sh
    wget https://raw.githubusercontent.com/lgsvl/apollo-5.0/simulator/scripts/apollo_base.sh -O scripts/apollo_base.sh

    bash docker/scripts/runtime_start.sh
    bash docker/scripts/runtime_into.sh

    The modules/map/data directory is used as a volume for maps, you can either use the maps included
    in `simulator` branch of LGSVL Apollo 5.0 [here](https://github.com/lgsvl/apollo-5.0/tree/simulator/modules/map/data)
    or download individual HD Maps as described in the next section.

### Adding an HD Map

* The default maps have their HD map files included in the [LGSVL Branch of Apollo 5.0](https://github.com/lgsvl/apollo-5.0/).

* More map AssetBundles and HD maps are available on our [content website](https://content.lgsvlsimulator.com/maps/).

* If you want to add a new HD map to Apollo 5.0 using our simulator, following steps are needed:
    - Export map annotations as `base_map.bin` file for Apollo 5.0 by following [Export Map Annotations](https://www.lgsvlsimulator.com/docs/map-annotation/#export-map-annotations)
    - Create a new map folder under `APOLLO_ROOT/modules/map/data/` and put `base_map.bin` in the folder.
    - Inside Apollo docker container
        - `cd /apollo`
        - Generate Apollo required map files
            - `generate_map.sh YOUR_MAP_FOLDER_NAME`
        - You need to restart dreamview to refresh the map list
            - `bootstrap.sh stop && bootstrap.sh`

## Launching Apollo alongside the simulator

Here we only describe only a simple case of driving from point A to point B using Apollo and the simulator. 

To launch Apollo, first launch and enter a container as described in the previous steps.

* To start Apollo:

        bootstrap.sh

    Note: you may receive errors about dreamview not being build if you do not run the script from the `/apollo` directory.

* Launch bridge (inside docker container):

        bridge.sh

* Run the LG SVL Simulator outside of docker. See instructions in the [simulator repository](https://www.lgsvlsimulator.com/docs/getting-started/#simulator-instructions)
    - Create a simulation with `BorregasAve` map and the `Lincoln2017MKZ (Apollo 5.0)` vehicles.
        - Next to the vehicle dropdown, enter the IP of the machine running Apollo and the port number of the bridge (default is 9090).
            - Example: Apollo is running on the same computer as the simulator. Bridge connection string will be `localhost:9090`.
    - (optional) Enable Traffic and Pedestrians.


* Open Apollo dreamview in a browser by navigating to: `localhost:8888`
    - Select the `Lincoln2017MKZ` vehicle and `BorregasAve` map in the top right corner.
    - Open the **Module Controller** tab (on the left bar).
    - Enable **Localization**, **Transform**, **Perception**, **Traffic Light**, **Planning**, **Prediction**, **Routing**.
    - Navigate to the **Route Editing** tab.
    - Select a destination by clicking on a lane line and clicking **Submit Route**.
    - Enable **Control** in **Module Controller** tab.
    - Watch the vehicle navigate to the destination.
    - To stop the docker container run the `dev_start.sh stop` script in `apollo/docker/scripts` in a new terminal (not in the docker container).
