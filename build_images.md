### Build an image from a Dockerfile

The docker build command builds Docker images from a Dockerfile and a “context”. A build’s context is the set of files located in the specified PATH or URL

Syntax is
```
docker build [OPTIONS] PATH | URL | -
```

The URL parameter can refer to three kinds of resources: Git repositories, pre-packaged tarball contexts and plain text files.

**Git repositories**

For example, run this command to use a directory called docker in the branch container:
```
docker build https://github.com/docker/rootfs.git#container:docker
```

**Tarball contexts**

If you pass an URL to a remote tarball, the URL itself is sent to the daemon:
```
docker build http://server/context.tar.gz
```
 
**Text files**
Instead of specifying a context, you can pass a single Dockerfile in the URL or pipe the file in via STDIN. To pipe a Dockerfile from STDIN:
```
docker build - < Dockerfile
```

Images can be build in many ways:
```
docker build - < Dockerfile

docker build  -t flaskapp:v1 -f- ./ < Dockerfile

docker build -t flaskapp .

docker build -t flaskapp:v2 .

docker build -t flaskapp:latest . -f ./Dockerfile
```

Typical keywords in a Dockerfile
* FROM -> It can be a parent image like ubuntu or base image like SCRATCH

* RUN -> The commands to be run inside the parent/base image to add more pacakges etc

* WORKDIR -> Set the working dir inside the image. It creates a new dir if not there.

* COPY -> Copy files and dirs from the host dir (typically current host dir) to some location in the image

* EXPOSE -> the port to be exposed from the container to the host

* ENTRYPOINT -> The ENTRYPOINT specifies a command that will always be executed when the container starts

* CMD -> The CMD specifies arguments that will be fed to the ENTRYPOINT.

Docker has a default entrypoint which is /bin/sh -c but does not have a default command.

When you run docker like this: docker run -i -t ubuntu bash the entrypoint is the default /bin/sh -c, the image is ubuntu and the command is bash.

The command is run via the entrypoint. i.e., the actual thing that gets executed is /bin/sh -c bash.

All dockerfiles may not have ENTRYPOINT or CMD. But, for an image to be runnable without additional docker run command line arguments, you must specify an ENTRYPOINT or CMD. Else, you will get error 'No command specified'.


They both specify programs that execute when the container starts running, but:

- CMD commands are ignored (or say overidden) by Daemon when there are parameters stated within the `docker run` command.

- ENTRYPOINT instructions are not ignored but instead are appended as command line parameters by treating those as arguments of the command.


**Adding CMD to unbuntu image**
When you run below, it will create`a new ubuntu image with CMD added to it.
```
echo "FROM ubuntu:trusty" > dockerf
echo "CMD ping localhost" >> dockerf

docker build -t ubuntu_v1 . -f dockerf
```

Now, run the container. It will keep running ping continuously.
```
docker run -it ubuntu_v1
```

Next, you can overide the CMD of `ping` by passing your own command. It will display container id which is the default hostname of a container.
```
docker run ubuntu_v1 hostname
```

The default ENTRYPOINT can also be overridden but it requires the use of the --entrypoint flag
```
docker run -it --entrypoint hostname ubuntu_v1
```

**Adding ENTRYPOINT and CMD to unbuntu image**
When you run below, it will create`a new ubuntu image with CMD added to it.
```
echo "FROM ubuntu:trusty" > dockerf
echo 'ENTRYPOINT ["/bin/ping"]' >> dockerf
echo 'CMD ["localhost"]' >> dockerf

docker build -t ubuntu_v2 . -f dockerf
```

Now, run the container. It will keep running ping continuously.
```
docker run -it ubuntu_v2
```

Next, you can overide the CMD of `localhost` by passing your own command. It will beging to ping that host.
```
docker run -it ubuntu_v2 bing.com
```

The default ENTRYPOINT can also be overridden but it requires the use of the --entrypoint flag
```
docker run --entrypoint hostname ubuntu_v1
```

**Adding CMD with multiple commands to unbuntu image**
When you run below, it will create`a new ubuntu image with CMD added to it.
```
echo "FROM ubuntu:trusty" > dockerf
echo 'CMD ["/bin/ping", "localhost"]' >> dockerf

docker build -t ubuntu_v3 . -f dockerf
```

Now, run the container. It will keep running ping continuously.
```
docker run ubuntu_v3
```

Next, you can overide the CMD by passing your own command. It will display container id which is the default hostname of a container.
```
docker run -it ubuntu_v3 hostname
docker run -it ubuntu_v3 bash
```

The default ENTRYPOINT can also be overridden but it requires the use of the --entrypoint flag
```
docker run -it --entrypoint hostname ubuntu_v3
```

Another variation
```
# no entrypoint
docker run ubuntu /bin/cat /etc/passwd

# with entry point, emulating cat command
docker run --entrypoint="/bin/cat" ubuntu /etc/passwd
```

Refer to below repos for more examples and follow the README there:

**Python Flask app example**
https://github.com/manojkmgit/docker_build_compose_flaskapp

**PHP wenn app example**
https://github.com/manojkmgit/docker_build_compose_phpapp


**More commands**
Create a new image from a container's changes
```
docker commit <container-id> mynewimage
```

Save one or more images to a tar archive (streamed to STDOUT by default)
```
docker save ubuntu_v3 > /tmp/ubuntu_v3.tar

docker save -o ubuntu.tar ubuntu:latest

docker image save ubuntu_v3 -o ubuntu_v3.img
```

Load an image from a tar archive or STDIN
```
docker load < /tmp/mynewimage.tar
```
