#### Docker Swarm Stack Deployment

https://docs.docker.com/engine/swarm/stack-deploy/

When running Docker Engine in swarm mode, you can use docker stack deploy to deploy a complete application stack to the swarm. The deploy command accepts a stack description in the form of a Compose file.

If you have an application which has multiple layers like web, app, database etc, then you should use docker compose to define all the layers (called as services in Docker Swarm parlance) in a single compose file. This way all layers can be managed together.

We are going to do following:
- Create a sample application which counts the number of visits to our portal.
- Verify the application locally.
- Push the images to registry.
- Finally, Deploy the application stack on the swarm.

##### Create the application

```ruby
git clone https://github.com/cloudxlab/docker_swarm_stack_demo
cd docker_swarm_stack_demo
```

This application has a Flask app in file app.py which does increment a counter on each portal access and stores the counter in the Redis in-memory data structure store.

Then, there is Dockerfile to build the image for the Flask application. We also have docker-compose.yml which defines the stack comprising of Flask app and Redis data store.

##### Verify the application locally
Bring up the stack locally.

```ruby
docker-compose up --build -b
docker-compose ps
```

Verify the application.
```ruby
curl localhost:5001
curl localhost:5001
curl localhost:5001
curl localhost:5001
curl localhost:5001
```

You should see an output like below which confirms that the portal visit counter is working.
```
ubuntu@ip-172-31-44-153:~/docker_swarm_stack_demo$ curl localhost:5001
Hello There World Bye! You have come here now 1 times.
ubuntu@ip-172-31-44-153:~/docker_swarm_stack_demo$ curl localhost:5001
Hello There World Bye! You have come here now 2 times.
ubuntu@ip-172-31-44-153:~/docker_swarm_stack_demo$ curl localhost:5001
Hello There World Bye! You have come here now 3 times.
ubuntu@ip-172-31-44-153:~/docker_swarm_stack_demo$ curl localhost:5001
Hello There World Bye! You have come here now 4 times.
ubuntu@ip-172-31-44-153:~/docker_swarm_stack_demo$
```

Now, bring the app down.
```ruby
docker-compose down --volumes
```

##### Push the images to registry
You can push the images to docker registry or any private registry as usual. But, docker compose provides an integrated way of doing this. Notice the image clause `image: manojkmhub/flask_stack_app:latest` in the docker-compose.yml

Because of this clause, when the app stack was brought up earlier, it already created the image tagged to the required docker hub. If you want to tag to a private registry, then image clause can be `image: 127.0.0.1:5000/flask_stack_app:latest`.

Image is already built, you can now push very easily.

```ruby
docker login

docker-compose push
```

##### Deploy the stack in swarm
Now that images are in the registry, we can deploy the stack in the swarm which will pull the images and start the containers on different nodes as decided by the swarm. 

The last argument is a name for the stack. Each network, volume and service name is prefixed with the stack name.

```ruby
docker stack deploy --compose-file docker-compose.yml swarm_stack_demo
```

Verify the services in the stack
```ruby
docker stack ls

docker stack services swarm_stack_demo

docker stack ps swarm_stack_demo
```

If need to see logs of any of the services, run this.
```ruby
docker service logs swarm_stack_demo_redis_host
docker service logs swarm_stack_demo_web
```

Verify the application running in the swarm.
```ruby
curl localhost:5001
curl localhost:5001
curl localhost:5001
```

Verify the swarm capability of the application by accessing the app pointing to any of the nodes.
```ruby
for host in `docker node ls --format '{{.Hostname}}'`
do
echo "Curling from $host"; echo ""
curl $host:5001
echo ""
done
```

Note that, you can access the app from any of the nodes in the swarm irrespective of whether the services are running on that node or not.

It is made possible by Docker’s built-in routing mesh.

If you have deployed this example on AWS EC2 or cloud machines, then you can access the application on public IP too over the internet.

Once tested, you can remove the stack.
```ruby
docker stack rm swarm_stack_demo
```
