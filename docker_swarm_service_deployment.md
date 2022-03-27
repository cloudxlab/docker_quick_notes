#### Docker Swarm Service Deployment

##### Deploy the services
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

##### Scale the services

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

##### Delete the servics
```ruby
docker service rm alpine_app
```

##### Update the service in rolling fashion
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

##### Rebalance the services
You can drain a nodes which will in effect rebalance the services among remaining active nodes.

Nodes can be in one of three availability states. ACTIVE, DRAIN, PAUSED. Default is ACTIVE. You can change the state from ACTIVE to DRAIN and vice versa.

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
