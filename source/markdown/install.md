# Installation Steps

1. Install dependencies
2. Build images or fetch images
3. Configure parts
4. Run each part

# Dependencies

List of dependencies for this project:
- Linux host OS
- [Docker](https://docs.docker.com/engine/install/)
- [docker-compose](https://docs.docker.com/compose/install/)

Please consult with your distribution for more information on how to install these for your machine.

For help regarding Docker, please visit [docker documentation](https://docs.docker.com/).

Each module has multiple dependencies; these are handled by the docker files when the images are built. No user interaction is required to install these. To see what those dependencies are, please check each module's docker file.

## Running Docker As A Service
It is recommended to run the Docker service at boot time. Else one will be responsible for starting the docker service manually before running this project.

Most modern Linux distributions use `systemd` as a service manager; the following commands apply. If your system uses `SystemV` please look [here](https://www.digitalocean.com/community/tutorials/how-to-configure-a-linux-service-to-start-automatically-after-a-crash-or-reboot-part-2-reference)  for a brief tutorial.

Start docker service manually now
```bash
sudo systemctl start docker
```

Start Docker at boot
```bash
sudo systemctl enable docker
```
To check the status of the Docker service   
```bash
sudo systemctl status docker
```

<!--
Do not start Docker at boot
```bash
sudo systemctl disable docker
```

Running the Docker daemon manually also works, as long as the daemon is running when using this project.
-->
<!--
### Other Requirements

#### Linux
Linux machine is required to run this project. Even though Docker grants us the ability to port projects, in this case, our build procedure only supports Linux at the moment.

#### Docker group
It is recommended to add the user that will be running the wrapper scripts to the `docker` user group. This will allow the script to connect to the Docker socket without the need to elevate privileges (e.g. sudo). The wrapper scripts automatically detect whether or not the user is part of the group and executes commands accordingly. The `root` user can skip this section.
```bash
# login as the user that will be executing the wrapper scripts

# add the docker group if it doesn't exist
# check existence with
cat /etc/group | grep docker

# create group
sudo groupadd docker

# add the current user to the docker group
sudo usermod -aG docker $USER

```
For group changes to take effect, logout the user and log back in. Then check if it worked. 
```bash
# check if docker service is running 
sudo systemctl status docker
# start docker service if already not running
sudo systemctl start docker

# check if we can execute Docker without sudo
docker run hello-world
````
-->

# Building images
## Image names
The following names are useful if wishing to handle image building and container creation manually or when using the deprecated scripts: 
|Module|Names|
|:-:|-|
|priv-libs|cybexp-priv-libs|
|collector|cybexp-collector|
|query|cybexp-priv-query|
|KMS|cybexp-kms-priv|
|backend API|cybexp-priv-backend-api|

All modules depend on the priv-libs image. Please build this image first before building any other image. 

Since this project runs on Docker, images can also be exported and imported to new machines without running the building procedure at each machine.

Wrapper scripts are provided to handle environment variable assignment, image building, persistent storage mounting, port mapping, and container deployment. It is highly recommended to use these scripts. 

Alternatively, one can manually build all these images by using Docker's build procedure (docker-compose). If building manually, assign environment variables as specified in the `docker-compose.yaml` files. Also, it is necessary to mount volumes or bind mount a filesystem for data persistency and bind ports when running manually (without wrapper script).

Please build before running for the first time or when the code has been modified. Rebuilding is not necessary if the project has already been build or imported. Building is unnecessary when changing configuration files, as these are mounted at run time (also handled by wrapper scripts).

## Build base image
To build the base image (cybexp-priv-libs):   
```bash
cd <project-priv-libs>
sh ./build.sh
```
## Build Collector, KMS, Backend, Query-client

It is necessary to build before running each module for the first time. Use the wrapper script provided in each of the module's folders to build.   


Build the collector:   
```bash
cd <module-collector>
bash collector.sh --build --build-only
```
Build the KMS:   
```bash
cd <module-kms>
bash kms.sh --build --build-only
```

Build the backend:   
```bash
cd <module-backend>
bash backend.sh --build --build-only
```

Build the query client:   
```bash
cd <module-query>
bash query.sh --build --build-only
```
# Important Information About Data Persistence

The  `secrets` folder that is located at the root of each module is mounted at runtime. It holds the keys. Each module has one. Keys are only fetched from the KMS server if they are not found there. One can mount an encrypted LUKS filesystem to the `secrets` folder before running the module to store these keys in the underlying hardware medium securely.

Secrets and configurations are never included in the images as they are mounted to the container at runtime. This prevents accidentally including secrets and configurations in images and allowing for modification of the configurations without rebuilding. One can modify the configuration of a module and restart said container for the changes to take effect.

The modules that use MongoDB instance (created by the wrapper script automatically) have their storage mounted under the directory `db` at runtime for persistency purposes. This directory is located at the root of the module. This can be configured with the docker-compose.yaml file.

Both `secrets` and `db` folders are mounted at runtime. It is important to note that these contain sensitive information and all the data that should be persistent. Removing the containers using the `docker rm` command will delete all the data in them as well but will not delete the persistent storage as they are part of the host system (unless modified otherwise). Recreating the containers with the same mount points (default if unchanged, change under `docker-compose.yaml`) will create new containers and mount the same folders/data as the previous container.  This is useful when updating code without deleting the data. This allows to stop and rename the old container to make a new container with the same data. If the update went well, then maybe delete the old container with the old code. For more information about data persistency, see [Docker Storage documnetation](https://docs.docker.com/storage/), [Dcoker bind mount documentation](https://docs.docker.com/storage/bind-mounts/), [Docker Volumes documentation](https://docs.docker.com/storage/volumes/).


```{include} bind_interface.md
```

```{include} data.md
```

```{include} kms_install.md
```