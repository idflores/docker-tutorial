# Docker Tutorial

Got sick of using `virtualenv` for my Python scripts and wanted to continue my professional experience in microservice development. Love Docker!

## Notes
Note: in `Dockerfile` **command order is important**. For instance, if the container is behind a proxy, the commands
```Dockerfile
ENV http_proxy host:port
ENV https_proxy host:port
```
should be used before the call to `pip` so that packages installed from `requirements.txt` are installed with the correct environment context.

Note: [reference](https://docs.docker.com/compose/overview/) for `docker-compose.yml` file

#### Commands
```bash
docker build -t friendlyhello .
```
The `build` command must be run whenever a change is made to the source code. It's purpose is similar to `Makefile` in that it performs commands to build the Docker container from its source. More info on the definition of a Docker "container" can be found [here](https://docs.docker.com/get-started/#containers-vs-virtual-machines).

Note: the `-t` option allows us to address our container in later commands using a human readable name (not a hash id)

Note: the `.` argument is the context in which Docker should perform the build

```bash
docker run -d -p 4000:80 friendlyhello

# **For use with Docker Public Registry**
# runs a docker image from the Docker remote repository
docker run -d -p 4000:80 <username>/<repository>:<tag>
```
The `run` command takes the *current* image build and instantiates an active container. Remember to run the `build` command first if changes are made to the source of the Docker image and you want them reflected in container implementations.

Note: the `-d` option causes Docker to run the container *detached* or as a background process

Note: the `-p` option *exposes* port "4000" on the system and *couples* it with port "80" from the exposed Docker container port.

Note: `friendlyhello` is the name we gave our docker container explained in the `build` command

```bash
docker container ls
```
Lists the containers currently running. Shows the
- Container ID
- the IMAGE or "NAME" of the Docker image
- the COMMAND used to execute an instance of the IMAGE
- the CREATED time the container was instantiated
- the STATUS or the uptime
- the PORTS the container is connected (e.g. `0.0.0.0:4000->80/tcp`)

```bash
docker container stop <Container NAME or ID>
```
Stops the specified container.

```bash
docker tag image username/repository:tag
```
The `tag` command gives your Docker image an address to be referenced by on the Docker Public Registry.

Note: `image` is the name of the image (e.g. friendlyhello)

Example: `docker tag friendlyhello john/get-started:part2`

```bash
docker image ls
#or
docker images
```
Lists the images.

```bash
docker save -o /path/to/image.tar <image>
docker load -i /path/to/image.tar
```
The `save` and `load` command are used to same an image locally as a `.tar` archive allowing the image to be transferred to another host and loaded without the use of the Docker Public Registry.

[StackOverflow](https://stackoverflow.com/questions/24482822/how-to-share-my-docker-image-without-using-the-docker-hub) - *How to share my Docker-Image without using the Docker-Hub?*

[StackOverflow](https://stackoverflow.com/questions/23935141/how-to-copy-docker-images-from-one-host-to-another-without-via-repository) - *How to copy docker images from one host to another without via repository?*

```bash
docker login
docker push <username>/<repository>:<tag>
```
__*For use with Docker's public registry*__
After logging into the Docker Public Registry, the `push` command is similar to `git push` and pushes the image to the Docker Public Registry (note: this is not a replacement for `git`).

```bash
docker swarm init
docker swarm join
docker swarm leave --force
```
The `swarm init` command enables Docker into "Swarm Mode" and makes the current machine Docker is running on a Swarm Manager.
Generally, the `swarm leave` command performs the opposite task.
Run `swarm join` command on other machines to have them join the current Swarm as "workers".

```bash
docker stack deploy -c docker-compose.yml <service_name>
docker stack rm <service_name>
```
Generally, the `stack deploy` command is equivalent to the combination of `docker build` and `docker run` for services. Instantiates a service stack that models the definition in the YAML file.
The `stack rm` command is, generally, equivalent to `docker container stop` but for a service.

Note: the `-c` option allows for a Docker compose YAML file to be specified.

Note: `<service_name>` is the name of the service we're instantiating (not the name of the image).

```bash
docker service ls
```
Lists all currently running services.

```bash
docker service ps -q <service_name>
```
Lists the "tasks" currently running on the service.

Note: a "task" is a image instantiation on the service

Note: the `-q` option denotes "quiet mode" and only outputs the Container IDs of the tasks running on the specified servie.

```bash
docker container ls
```
Lists all containers currently running on the machine. Note, this is not filtered by service; for that, use `docker service ps` instead.

##### docker-machine
```bash
docker-machine create -d virtualbox <machine_name>
```
Creates a virtual machine using the VirtualBox driver and in the Docker environment.

```bash
docker-machine env <machine_name>
```
Lists basic environment information about the specified virtual machine.

```bash
docker-machine ssh <machine_name>
docker-machine ssh <machine_name> "<command>"
```
The *first* enters an ssh shell of the specified virtual machine. The *second* performs the command on the specified virtual machine.

```bash
docker-machine ssh <machine_name> "docker swarm init"
docker-machine ssh <machine_name> "docker swarm join <token> <ip>:<port>"
docker-machine ssh <machine_name> "docker stack deploy -c <file> <service_name>"
```
The *first* elevates the specified virtual machine to Swarm Manager. The *second* causes the specified virtual machine to join the Swarm using the `<token>`, `<ip>` and `<port>` of the Swarm Manager given during the output of the *first*. The *third* deploys the specified services to the specified virtual machine.

```bash
docker-machine ssh <machine_name> "docker node ls"
```
Lists all nodes currently registered with the Swarm implicitly indicated by the specified virtual machine.

```bash
eval $(docker-machine env <machine_name>)
eval $(docker-machine env -u)
```
Generally, the *first* causes the context of the current shell to the ssh shell of the specified virtual machine. This is different from entering the shell directly using `docker-machine ssh <machine_name>`. By changing only the context, we now have access to local machine files (e.g. `docker-compose.yml`). The *second* unsets that context back to the local machine.

If a context is not set, use `docker-machine scp <file> <machine_name>:~` to copy files from the local machine to the virtual machine.

```bash
docker-machine ssh <machine_name> "docker swarm leave --force"
docker-machine stop $(docker-machine ls -q)
docker-machine rm $(docker-machine ls -q)
```
The *first* removes the specified worker virtual machine from the Swarm. The *second* stops all running virtual machines. The *first* deletes all virtual machines including their disk images.



## Development Environment
#### Tools
+ Docker v17.12.0-ce
  + Python v2.7-slim (Docker Image)
    + Flask v0.12.2
    + Redis v2.10.6

#### Machine
+ MacOS High Sierra 10.13.2
+ [Atom.io](https://atom.io) Editor
+ Safari
