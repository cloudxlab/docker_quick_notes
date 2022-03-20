### Docker Networking

#### Network drivers
Docker’s networking subsystem is pluggable, using drivers. Several drivers exist by default, and provide core networking functionality:

- bridge - default network driver, used for standalone containers

- host - used for standalone containers, remove network isolation between the container and the Docker host, and use the host’s networking directly

- overlay - Overlay networks connect multiple Docker daemons together and enable swarm services to communicate with each other. You can also use overlay networks to facilitate communication between a swarm service and a standalone container, or between two standalone containers on different Docker daemons. This strategy removes the need to do OS-level routing between these containers.

- ipvlan: IPvlan networks give users total control over both IPv4 and IPv6 addressing. The VLAN driver builds on top of that in giving operators complete control of layer 2 VLAN tagging and even IPvlan L3 routing for users interested in underlay network integration.

- macvlan: Macvlan networks allow you to assign a MAC address to a container, making it appear as a physical device on your network. The Docker daemon routes traffic to containers by their MAC addresses. Using the macvlan driver is sometimes the best choice when dealing with legacy applications that expect to be directly connected to the physical network, rather than routed through the Docker host’s network stack.

- none: For this container, disable all networking. Usually used in conjunction with a custom network driver. none is not available for swarm services.

- Network plugins: You can install and use third-party network plugins with Docker. These plugins are available from Docker Hub or from third-party vendors. See the vendor’s documentation for installing and using a given network plugin.

#### List all networks a container belongs to
docker inspect -f '{{range $key, $value := .NetworkSettings.Networks}}{{$key}} {{end}}' [container]

#### List all containers belonging to a network by name
docker network inspect -f '{{range .Containers}}{{.Name}} {{end}}' [network]

##### Connect a running container to a network
docker network connect [network] [container]

You can also specify a network when you start a container with the --network (or --net) flag as follows:

##### Connect a container to a network when it starts
docker run --network [network] [container]

##### With containerA already running, test if containerA can connect to containerB by using its name
docker exec [containerA] ping [containerB] -c2

#### Ping between 2 containers on default bridge network and user created bridge network

```
sudo apt install net-tools
ifconfig
```

You will see bridge0 or docker0. That is the default bridge network.

The Docker bridge driver automatically installs rules in the host machine so that containers on different bridge networks cannot communicate directly with each other.

You can view the iptables data using below command.
```
sudo iptables -S
```

View all docker networks
```
docker network ls
```

Start two containers and study the network related details.
```
docker run -d --name nginx1 -p 8001:80 nginx:latest
docker run -d --name nginx2 -p 8002:80 nginx:latest

docker container inspect nginx1
docker container inspect nginx2
```

Both containers will be running on default bridge network. IPs may be like 172.17.0.2, 172.17.0.3 etc.

```
docker network inspect bridge | grep -i docker.network
docker network inspect bridge
docker inspect -f '{{range $key, $value := .NetworkSettings.Networks}}{{$key}} {{end}}' nginx1
docker inspect -f '{{range $key, $value := .NetworkSettings.Networks}}{{$key}} {{end}}' nginx2
docker network inspect -f '{{range .Containers}}{{.Name}} {{.IPv4Address}} {{end}}' bridge
```

Install ping inside both containers.

```
docker exec -it nginx1 bash -c "apt update"
docker exec -it nginx1 bash -c "apt install iputils-ping -y"

docker exec -it nginx2 bash -c "apt update"
docker exec -it nginx2 bash -c "apt install iputils-ping -y"
```

Check the ping status from each container to other one.
```
docker exec -it nginx1 ping nginx2 # ping by container name will not work as containers are on default network
docker exec -it nginx2 ping nginx1 # ping by container name will not work as containers are on default network

docker exec -it nginx1 ping 172.17.0.3 # ping by IP address will work
docker exec -it nginx2 ping 172.17.0.2 # ping by IP address will work
```

Now, create a bridge network manually.
```
docker network create --driver=bridge mynet
docker network inspect mynet
```

Connect the new network with the containers. Containers will still have old network.
```
docker network connect mynet nginx1
docker network inspect -f '{{range .Containers}}{{.Name}} {{.IPv4Address}} {{end}}' mynet
docker inspect -f '{{range $key, $value := .NetworkSettings.Networks}}{{$key}} {{end}}' nginx1
docker inspect -f '{{range $key, $value := .NetworkSettings.Networks}}{{$key}} {{end}}' nginx2
```

At this point of time, nginx1 can ping to old IP of nginx2 but nginx2 can not ping to new IP of nginx1 because nginx2 still has only default bridge network.

```
docker network connect mynet nginx2
docker network inspect -f '{{range .Containers}}{{.Name}} {{.IPv4Address}} {{end}}' mynet
docker inspect -f '{{range $key, $value := .NetworkSettings.Networks}}{{$key}} {{end}}' nginx1
docker inspect -f '{{range $key, $value := .NetworkSettings.Networks}}{{$key}} {{end}}' nginx2
```

At this point of time, nginx1 can ping to old and new IP of nginx2, and nginx2 can ping to old and new IP of nginx1 because nginx1 and nginx2 both are now part of `bridge` and `mynet` networks.

But most important change now is that containers can ping each other using container name too.
```
docker exec -it nginx1 ping nginx2
docker exec -it nginx2 ping nginx1
```

If you want, you can now disconnect the containers from the default bridge network but you may need to first stop the containers, and then run the containers again with user created network.
```
docker container stop nginx1
docker container stop nginx2
docker network disconnect bridge nginx1
docker network disconnect bridge nginx1

docker container rm nginx1
docker container rm nginx2

docker run -d --name nginx1 --network mynet -p 8001:80 nginx:latest
docker run -d --name nginx2 --network mynet -p 8002:80 nginx:latest
```

#### Ping between 2 containers on host network

```
docker container rm nginx1 --force
docker container rm nginx2 --force

#port publish using -p is NA for host network
#here, using ubuntu for other container as 80 port is used on the host by nginx1 and another container can not be started
docker run -d --name nginx1 --network=host nginx:latest
docker run -dit --name ubuntu2 --network=host ubuntu:latest

docker exec -it nginx1 bash -c "apt update"
docker exec -it nginx1 bash -c "apt install iputils-ping -y"

docker exec -it ubuntu2 bash -c "apt update"
docker exec -it ubuntu2 bash -c "apt install iputils-ping -y"

docker exec -it nginx1 ping localhost
docker exec -it ubuntu2 ping localhost

docker network inspect -f '{{range .Containers}}{{.Name}} {{.IPv4Address}} {{end}}' host
docker inspect -f '{{range $key, $value := .NetworkSettings.Networks}}{{$key}} {{end}}' nginx1
docker inspect -f '{{range $key, $value := .NetworkSettings.Networks}}{{$key}} {{end}}' nginx2
```
