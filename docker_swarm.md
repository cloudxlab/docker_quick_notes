### Docker Swarm

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
```ruby
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
docker node update --availability drain master1
```

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

##### Deploy a service to the swarm

Go the master node and run below to start 1 instance of a given container.
```ruby
docker service create --replicas 1 --name alpine_app alpine ping docker.com

# --detach Exit immediately instead of waiting for the service to converge
docker service create --detach=false --name phpwebapp --publish published=8080,target=80 manojkmhub/phpapp
```

Verify the service.
```ruby
docker service ls

docker service ps alpine_app

docker service ps alpine_app --format '{{.Node}}'
```

Inspect the service.
```ruby
docker service inspect --pretty alpine_app

docker service inspect alpine_app
```

Check the nodes where services are running.
```ruby
docker service ps alpine_app
```

Go the node where the service is running and verify.
```ruby
docker container ls

docker ps
```

##### Scale the service in the swarm

```ruby
docker service scale alpine_app=5
```

Check the service details now. It is possible that services got started on master node too.
```ruby
docker service ls

docker service ps alpine_app
```

You can scale down too.
```ruby
docker service scale alpine_app=1
```

You can scale down to 0 as well. When you do this, service is not removed from swarm configuration. It is just that no service is running.
```ruby
docker service scale alpine_app=0

docker service ls
```

You can scale up directlty from 0 to desired value.
```ruby
docker service scale alpine_app=2
```

##### Delete the service running on the swarm
```ruby
docker service rm alpine_app
```

##### Apply rolling updates to a service
Services can be updated in many ways. See full list as below.
```ruby
docker service update --help
```

Here, we will first create 4 containers of redis of some lower version. When we later upgrade the redis version, it will update the services one by one at interval defined by `--update-delay`.

By default, when once a service is updated, it will return RUNNING status, then next service will be updated after the given delay. If any update fails, it will pause there.

```
--update-delay
    seconds Ts
    minutes Tm
    hours Th

Example:
10s
10m30s -> 10 minute 30 seconds delay
```

Now, create the containers with lower version.
```ruby
docker service create \
  --replicas 4 \
  --name redis_app \
  --update-delay 20s \
  redis:3.0.6
  ```

Verify the services.
```ruby
docker service ps redis_app

docker service inspect --pretty redis_app
```

Now, update the image version.
```ruby
docker service update --image redis:3.0.7 redis_app
```

Verify the previous and new specs of the service.
```ruby
docker service inspect --format '{{.PreviousSpec.TaskTemplate.ContainerSpec.Image}}' redis_app

docker service inspect --format '{{.Spec.TaskTemplate.ContainerSpec.Image}}' redis_app
```

Another example of update:
```ruby
docker service update --image redis:3.0.7 --update-delay 15s --update-order "start-first" redis_app
```

In next example, we will add an env var to the service.
```ruby
docker service update --detach=false --env-add MY_NODE_ID="{{.Node.ID}}" --update-delay 20s redis_app

docker service ps redis_app
```

Verify the env var was updated.
```ruby
docker service ps --format 'table {{.Name}}\t{{.Node}}\t{{.CurrentState}}' redis_app

# run this on the node where container has been started recently
docker container inspect --format '{{json .Config.Env}}' $(docker container ls -lq) | jq '.'
```

To restart a paused update, run docker service update <SERVICE-ID>. For example:
```ruby
docker service update redis_app
```

To rollback the service update:
```ruby
docker service update --detach=false --rollback redis_app
```

##### Drain a node on the swarm
Nodes can be in one of two availability states. ACTIVE and DRAIN. Default is ACTIVE. You can change the state from ACTIVE to DRAIN and vice versa.

When you change the node from ACTIVE to DRAIN state, any container running on that node as part of swarm set up will be stopped and launched on any ACTIVE nodes.

Check the node status and drain one of the nodes.
```ruby
docker node ls

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
