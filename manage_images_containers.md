### Manage docker - images and containers

To view all options of various commmands, you can run commands like below.
```
docker --help
docker run --help
docker image --help
docker container --help
```

Most of the time, you will be running commands like this.
```
docker image [COMMAND] <image_id/image_name>[:tag]
docker container run [OPTIONS] IMAGE[:TAG|@DIGEST] [COMMAND] [ARG...]
docker container [COMMAND] <container_id/container_name>
```

If you don't give value of 'TAG', then it defaults to 'latest'. If the image in the history doesn't have a version with tag of 'latest', then you must give the tag value while accessing the images.

When you execute 'docker run', Docker will pull the image from the Docker Hub registry (https://hub.docker.com/) or any other configured registry, if already available on the local machine and then, run a container using that image.

If the image doesn't have any foreground process added to the Docker Entrypoint and you run the container in non-interactive mode, then container will run and finish off quickly.

The command 'docker run' outputs the container id when goes for a run in detached mode.

At each step, you can run below commands to list running and all (stopped/running) containers.
```
docker container ls
docker container ls -a
docker ps
docker ps -a
```

You can run a basic container like below which will pull the image, run the container and finish off.
```
docker run hello-world
docker container run hello-world
```

Everything starts from an image, so let's explore images first.

**Explore images**
Docker images are read-only templates used to build then containers. An image contains the source code, libraries, dependencies, tools, and other files needed for an application to run. Image itself can not run. A container is build in runtime using the image as a template, and then the container is run.

Pull the image but not run a container for it.
```
docker pull ubuntu
docker pull ubuntu:latest
docker image pull ubuntu
docker image pull ubuntu:latest
```

After pulling the image, if you can just create the container without running it using create command. And then you can start the container.
```
docker create --name myflaskapp flaskapp:latest
docker start -i myflaskapp
```

The command 'docker run' does 'create' and 'start' in a single command.

Run the containers without pulling the image.
```
docker container run --pull=never ubuntu
docker container run --pull=never ubuntu:latest
docker container run --pull=never ubuntu:bionic
```

Pull the image if missing and run it.
```
docker container run --pull=missing ubuntu
```

Pull the image always and run it.
```
docker container run --pull=always ubuntu
```

List all the images on the local machine
```
docker image ls
```

Inspect the image to see location of the image on the local machine
```
docker image inspect ubuntu

docker image inspect -f '{{.GraphDriver.Data}}' ubuntu | sed 's/[ ]/\n/g;s/[\[]/[\n/g;s/[]]/\n]/g;'
```

Inspect using jquery
```
sudo apt -y install jq

docker image inspect -f '{{json .}}' ubuntu:latest | jq -r '. | {Id: .Id, Digest: .Digest, RepoDigests: .RepoDigests, Labels: .Config.Labels}'

docker image inspect -f '{{json .}}' ubuntu:latest | jq -r '. | {GraphDriver: .GraphDriver}'
```

Inspect to get container IP address
```
docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' ubuntu
```

Remove the container automatically when it stops
```
docker container run --rm ubuntu
```

A container by default run in foreground mode. If the container's main process finishes, it exits. If the image being used for the container doesn't have any long running main process like a web server, listener etc, then the container will exit immediately after start, by default. But, you start the container in interactive mode, then the container will keep running. You can also add some main process to the container during start, then also, the container will keep running.

For example, the main process in an ubuntu image is bash. When an ubuntu container starts, bash program finishes off immediately and hence, the ubuntu container also stops immediately.

You can keep it running using any of the ways given below.

**Method 1 - Use interactive mode as explained later**

**Method 2 - Add some long running main process to the container**
```
docker container run ubuntu tail -f /dev/null

docker container run ubuntu bash -c "while true; do sleep 1; done"

docker container run -d ubuntu bash -c "while true; do sleep 1; done"

docker container run -d ubuntu bash -c "trap : TERM INT; sleep infinity & wait"

docker run -d busybox top
```

Run containers in **detached/background** mode. In this case too, the container will keep running only if the container has some main process to run for long time. Else, even with -d option, it will stop immediately.
```
docker container run -d nginx
docker container ls
```

Run containers in **foreground** mode. This is the default mode. In this mode, docker run can start the process in the container and attach the console to the process’s standard input, output, and standard error. It can even pretend to be a TTY (this is what most command line executables expect) and pass along signals. All of that is configurable:
```
-a=[]           : Attach to `STDIN`, `STDOUT` and/or `STDERR`
-t              : Allocate a pseudo-tty
--sig-proxy=true: Proxy all received signals to the process (non-TTY mode only)
-i              : Keep STDIN open even if not attached
```

If you don't give '-t' option, it will work normally but you will not see any prompt (PS1) inside in the container terminal.

If you do not specify -a then Docker will attach to both stdout and stderr . You can specify to which of the three standard streams (STDIN, STDOUT, STDERR) you’d like to connect instead, as in below. It will start the container and open a terminal into the container attached to the init process PID 1 of the container.
```
docker container run -a stdin -a stdout -i -t --name myubuntu1 ubuntu /bin/bash
    ps -ef
	echo $$
```

If you give 'exit' command, then the container itself will stop because you were attached to init process PID 1 of the container.

To come out without stopping the container, press Ctl+p Ctl+q

Now, you can get into the container again in 2 ways:
1. Attach to the init process PID 1

```
docker attach <container_id/container_name>
  ps -ef
  echo $$
```

Then, come out safely, press press Ctl+p Ctl+q

2. Open a terminal into the container which will spawn in a new process and attach to that.
```
docker exec -it <container_id/container_name> /bin/bash
  ps -ef
```

Then, you can come out safely using exit or Ctl+p Ctl+q. But using Ctl+p Ctl+q on non-init process PID 1 will leave the additional process running inside the container.

In short, you can run the images in -di or -dit mode which starts the container in detached mode as well as keeps STDIN open even after detaching.
```
docker run -dit --name myubuntu1 ubuntu
```

Then, you can attach or connect to the the container.


```
docker attach myubuntu1
    echo $$
```

```
docker exec -dit myubuntu1 /bin/bash
    echo $$
```

**Remove images**
You can not remove an image if there are containers running with the image.\
Remove the image if there are no containers in running or stopped status with this image.
```
docker image rm ubuntu
```

Remove the image if there are no containers in running but there may be stopped containers with this image.
```
docker image rm ubuntu --force
```

You can also remove all unused images using prune command.
```
docker image prune
docker image prune --force
```

**System wide prune**
```
docker system prune -a
docker system prune -f
docker system prune -af
```

**Explore containers**
A container is build in runtime using the image as a template, and then the container is run. Containers use their separate space on the disk. A container is a writable environment when it is running and any files modified inside the container will persist even if container is stopped.

You can view all options using below command.

```
docker container --help
```

You can perform actions of stop/start/kill/restart/exec/kill/logs and many more.


Run containers with more options. By default, hostname of the container is same as the container id. You can give custom hostname to each container when starting. You can map the port of host to that of container's using -p <host_port>:<container_port>

```
docker container run -d -p 8000:80 --name mynginx --hostname mynginxhost --rm nginx
docker container ls
curl localhost:8000
```

Allocate cpu/meomory and mount a host file/dir into the container using --mount option.
```
docker container run -dit \
-p 8000:80 \
--name mynginx \
--hostname mynginxhost \
--rm \
--memory 100M \
--cpus 1 \
--mount type=bind,source="$(pwd)"/data_dir,target=/app_data_dir \
nginx

docker container ls
docker exec -it mynginx bash
docker container exec -it mynginx bash

curl localhost:8000
```

Mount using -v or --volume option which is similar to --mount. But, with -v option, the target file/dir if not exists, will be created as a dir.
```
docker container run -dit \
-p 8000:80 \
--name mynginx \
--hostname mynginxhost \
--rm \
--memory 100M \
--cpus 1 \
-v "$(pwd)"/data_dir:/app_data_dir \
nginx

docker container ls
docker exec -it mynginx bash

curl localhost:8000
```

Mount in read-only mode.
```
docker container run -dit \
-p 8000:80 \
--name mynginx \
--hostname mynginxhost \
--rm \
--memory 100M \
--cpus 1 \
-v "$(pwd)"/data_dir:/app_data_dir:ro \
nginx

docker container ls
docker exec -it mynginx bash

curl localhost:8000
```

Run the container with custom env vars which will become inside the container.
```
docker container run -dit \
-p 8000:80 \
--name mynginx \
--hostname mynginxhost \
--memory 100M \
--cpus 1 \
-v "$(pwd)"/data_dir:/app_data_dir \
-e NAME=MK \
-e LOC=EARTH \
nginx

docker container ls
docker exec -it mynginx bash

curl localhost:8000
```


Run a command inside the container
```
docker exec -it mynginx hostname
docker exec -it mynginx env
docker exec mynginx env
docker exec -it mynginx bash -c "env"
docker exec -it mynginx sh -c "env"
```

Get specific env variables value from the running container
```
docker exec -it mynginx bash -c 'echo "$ENV_VAR"'
docker exec mynginx bash -c 'echo "$ENV_VAR"'
docker exec mynginx bash -c 'echo "$HOSTNAME"'
docker container exec mynginx env | grep HOSTNAME | cut -d'=' -f2

docker inspect -f \
   '{{range $index, $value := .Config.Env}}{{$value}} {{end}}' mynginx

docker inspect mynginx | jq '.[] | .Config.Env'

```


Run multiple commands.
```
docker exec -it mynginx bash -c "hostname && env"
```

Write to a file inside the container.
```
docker container exec -it mynginx bash -c "hostname && env > /tmp.txt"
```

You can pause/unpause/stop/start the containers. Stopping the container will keep the filesystem of the container intact and you can start the exact same container again.
```
docker stop mynginx # sends SIGTERM to the init process
docker stop mynginx -t 10s # first sends SIGTERM, then sends SIGKILL and kills explicitly if container fails to stop with in 10 seconds
docker start mynginx
docker restart mynginx
docker stats mynginx
docker pause mynginx
docker unpause mynginx

docker kill mynginx # sends SIGKILL to the init process or any different process
docker kill mynginx -s SIGTERM # override the signal to be sent to the init process
```

View running processes of a container
```
docker top mynginx
```

Run docker system wide commands
```
docker system --help
docker system df # Show docker disk usage
docker system events # Get real time events from the server
docker system info # Display system-wide information
docker system prune # Remove unused data
```

You can rename the container
```
docker container rename old_name new_name
```

You can update container config
```
docker container inspect 860cf09df8f9 | grep -i cpus
docker container update --cpus-cpus 1  860cf09df8f9
```

Inspect changes to files or directories on a container's filesystem
```
docker container diff 860cf09df8f9
```

**Remove containers**

You can remove a running container, in general. A container is removed automatically if it was started with --rm option. Else, you can remove manually using rm or prunue commands, similar to removing the images.

```
docker container rm mynginx
```

Remove the container forcefully even if it running.
```
docker container rm mynginx --force
```
