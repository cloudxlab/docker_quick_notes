### Docker Swarm Setup

https://docs.docker.com/engine/swarm/key-concepts/


Docker Swarm is the cluster management and orchestration tool which enables deployment and management of containers across multiple Docker daemons or hosts.

Swarm consists of one of more manager nodes and one or more worker nodes.

A given Docker host can be a manager, a worker, or perform both roles.

When you create a service, you define its optimal state (number of replicas, network and storage resources available to it, ports the service exposes to the outside world, and more). Docker works to maintain that desired state. 

#### Steps to set up Docker Swarm

##### Launch 3 EC2 machines

Launch 3 EC2 ubuntu or any OS machines in same security group. Below steps are tested with ubuntu OS.

Give them below tags:\
manager1\
worker1\
worker2

Allow below ports in the security group.

TCP port 2377 for cluster management communications\
TCP and UDP port 7946 for communication among nodes\
UDP port 4789 for overlay network traffic\

##### Install docker on all the machines.
For this, refer to docker install quick notes readme in the current repo or follow official docker documentation.

##### Init the swarm

Connect to the manager node. Get the private and run below command.

Get private IP of EC2 machine:
```ruby
curl http://169.254.169.254/latest/meta-data/local-ipv4
MANAGER_PRIVATE_IP=`curl http://169.254.169.254/latest/meta-data/local-ipv4`
echo $MANAGER_PRIVATE_IP
```

Get public IP of EC2 machine:
```ruby
curl http://169.254.169.254/latest/meta-data/public-ipv4
curl http://checkip.amazonaws.com
```

```ruby
docker swarm init --advertise-addr $MANAGER_PRIVATE_IP
```

You will see this ouput:
```
ubuntu@ip-172-31-42-57:~$ docker swarm init --advertise-addr $MANAGER_PRIVATE_IP
Swarm initialized: current node (k9xhttgzgcx5p37dzio9l8mhv) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-03d1q9kisw1ybd49z96oswc8v2sn3nw0cx44xe3u5qgvi0p169-06iuuwtdcez24xv1o6r86sup3 172.31.42.57:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.

ubuntu@ip-172-31-42-57:
```

View the status
```ruby
docker info | grep -i 'swarm\|manager\|node'

docker node ls

docker node inspect self --pretty
```

##### Add nodes to the swarm
Take the command from init output
```ruby
docker swarm join --token SWMTKN-1-03d1q9kisw1ybd49z96oswc8v2sn3nw0cx44xe3u5qgvi0p169-06iuuwtdcez24xv1o6r86sup3 172.31.42.57:2377
```

You should see output as
```
This node joined a swarm as a worker.
```

In case you don't have the join commmand, run this on master node and then run the generated command back here on the worker node
```ruby
docker swarm join-token worker
```

Repeat above steps on all worker nodes.

Verify the swarm from master node
```ruby
docker node ls
```

You will see something like:
```ruby
ubuntu@ip-172-31-42-57:~$ docker node ls
ID                            HOSTNAME           STATUS    AVAILABILITY   MANAGER STATUS   ENGINE VERSION
p8zsqts5n0xnvbo821w8hd3bi     ip-172-31-38-232   Ready     Active                          20.10.13
k9xhttgzgcx5p37dzio9l8mhv *   ip-172-31-42-57    Ready     Active         Leader           20.10.13
r0y8hvbn9tpek0m8v58r3ix4y     ip-172-31-42-222   Ready     Active                          20.10.13
ubuntu@ip-172-31-42-57:~$
```

More commands to view the status:
```ruby
docker node ls --format 'table {{.ID}} {{if .Self}}*{{else}} {{end}}\t{{.Hostname}}'

docker node ls --format 'table {{.ID}} {{if .Self}}current{{else}} {{end}}\t{{.Hostname}}\t{{.ManagerStatus}}'
```

Verify manager status.
```ruby
docker node inspect ip-172-31-42-57 --format "{{.ManagerStatus.Reachability}}"

docker node inspect ip-172-31-42-57 --format "{{.Status.State}}"
```

If you don't want the container to run on the master node, you can run now below on the master node.
```ruby
docker node update --availability drain `hostname`
```

Possible availability options are:
- active
- pause
- drain

##### Remove nodes from the swarm
Go to the node to be removed from the swarm. There, you have to run `leave` command.\
If leaving from any worker node:
```ruby
docker swarm leave
```

If leaving from any master node:
```ruby
docker swarm leave --force
```

After leaving, go to any of the master nodes and remove the ndoe from node list.
```ruby
docker node rm worker1
```

##### Drain a node on the swarm
Nodes can be in one of three availability states. ACTIVE, DRAIN, PAUSED. Default is ACTIVE. You can change the state from ACTIVE to DRAIN and vice versa.

When you change the node from ACTIVE to DRAIN state, any container running on that node as part of swarm set up will be stopped and launched on any ACTIVE nodes.

Check the node status, create services and drain one of the nodes.
```ruby
docker node ls

docker service create --replicas 4 --name alpine_app alpine ping docker.com

docker service ps redis_app

docker node update --availability drain ip-172-31-42-222

docker service ps redis_app --filter desired-state=running

docker service inspect redis_app
```

Make the node ACTIVE again. This will not rebalance the already running services across the nodes.
```ruby
docker node update --availability active ip-172-31-42-222
```

But if you scale up, it will rebalance.
```ruby
docker service ps redis_app --filter desired-state=running

docker service scale redis_app=5

docker service ps redis_app --filter desired-state=running
```

You can force the rebalance but it will stop and start the containers as needed.
```ruby
docker service update --force redis_app
```

##### Docker Swarm Networking

Docker Swarm setup creates two networks on each node: `docker_gwbridge` and `ingress`.

```ruby
docker network ls
```

```
ubuntu@ip-172-31-44-153:~/docker_swarm_stack_demo$ docker network ls
NETWORK ID     NAME              DRIVER    SCOPE
edf129da84b3   bridge            bridge    local
426089bd1a34   docker_gwbridge   bridge    local
be0572e71a04   host              host      local
xfx0b7crr85x   ingress           overlay   swarm
bb7e59b2abb8   none              null      local
ubuntu@ip-172-31-44-153:~/docker_swarm_stack_demo$
```

Irrespective of Swarm environment or otherwise, Overlay networks manage communications among the Docker daemons participating in the swarm.

In terms of Swarm, The `ingress` network is a special overlay network that facilitates load balancing among a service’s nodes.

The `docker_gwbridge` is a bridge network that connects the overlay networks (including the ingress network) to an individual Docker daemon’s physical network. In other words, it connects the individual Docker daemon to the other daemons participating in the swarm.By default, each container a service is running is connected to its local Docker daemon host’s docker_gwbridge network.

**Docker Swarm Routing Mesh**

https://docs.docker.com/engine/swarm/ingress/

Docker Engine swarm mode makes it easy to publish ports for services to make them available to resources outside the swarm. All nodes participate in an ingress routing mesh. The routing mesh enables each node in the swarm to accept connections on published ports for any service running in the swarm, even if there’s no task running on the node. The routing mesh routes all incoming requests to published ports on available nodes to an active container.

![Swarm Routing Mesh](https://docs.docker.com/engine/swarm/images/ingress-routing-mesh.png)


