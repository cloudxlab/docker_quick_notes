#### Docker Compose

Compose is a tool for defining and running multi-container Docker applications. With Compose, you use a YAML file to configure your application’s services. Then, with a single command, you create and start all the services from your configuration.

##### Using Compose is a three-step process:

**1.** Define your app's environment with a Dockerfile so it can be reproduced anywhere. If images are already built, then Dockerfile may not be needed.

**2.** Define the services of your app in docker-compose.yml. All those services run together in an isloated environment.

If you need to build an image in runtime, then this file will have `build` command which in turn will need presence of a Dockerfile in the current context. If you need to use already images, then this file will have `image` command. Depending on your app, you may have both or any or none of `build` and `image`.

**3.** Run `docker compose up` which starts and runs the app stack. Or, you can run `docker-compose up` using the docker-compose binary.

##### Install Docker Compose from the binary in Docker’s GitHub repository

```ruby
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

sudo chmod +x /usr/local/bin/docker-compose

sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose

sudo chmod 755 /usr/local/bin/docker-compose

docker-compose --version
```

Then, you write a docker-compose.yml file having the stack of the app services.

A typical file may look like:

```ruby
version: "3.9"  # optional since v1.27.0
services:
  web:
    build: .
    ports:
      - "8000:5000"
    volumes:
      - .:/code
      - logvolume01:/var/log
    links:
      - redis
  redis:
    image: redis
volumes:
  logvolume01: {}
```

You can now build and run the app stack. The option -d is to run in detached mode. The option -p is to give custom project name which will be prefixed to images and containers names.
Below are different variations of the commands to build the images and bring up the stack and bring down the stack as defined in docker compose file.
```ruby
docker-compose build
docker-compose -f docker-compose.yml -p "MyPhpApp" build
docker-compose -f docker-compose.yml -p "MyPhpApp" build --force-rm
docker-compose images
docker-compose -p "MyPhpApp" images
docker-compose up
docker-compose up -d
docker-compose -f docker-compose.yml -p "MyPhpApp" up -d
docker-compose config
docker-compose -f docker-compose.yml -p "MyPhpApp" up -d --force-recreate
docker-compose ps
docker-compose -f docker-compose.yml down
docker-compose -f docker-compose.yml -p "MyPhpApp" down
docker-compose -p "MyPhpApp" down
```

To uninstall Docker Compose if you installed using curl:
```ruby
sudo rm /usr/local/bin/docker-compose
```

To uninstall Docker Compose if you installed using pip:
```ruby
pip uninstall docker-compose
```

##### Scaling servics in docker compose
One port of the host can be mapped to only one container port. Once a given host port is occupied, creating more containers with same port mapping will not work. The services defined in docker compose file can be scaled up if a range of host ports is mapped to the same container port.

Example given below:
```ruby
version: '3'
services:
    docker-registry:
        image: registry:2
        restart: always
        volumes:
        - "/tmp/docker_registry:/var/lib/registry"
        ports:
        - "2000-2005:2000"
        restart: always
```

Start the containers.
```ruby
docker-compose -f docker-compose.yml up -d --scale docker-registry=4
```

It will deploy the containers on any host ports between the given range.

##### Additional features:

You can use override compose files or use extend.
