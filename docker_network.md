https://docs.docker.com/network/

# List all networks a container belongs to
docker inspect -f '{{range $key, $value := .NetworkSettings.Networks}}{{$key}} {{end}}' [container]

# List all containers belonging to a network by name
docker network inspect -f '{{range .Containers}}{{.Name}} {{end}}' [network]

# Connect a running container to a network
docker network connect [network] [container]

You can also specify a network when you start a container with the --network (or --net) flag as follows:

# Connect a container to a network when it starts
docker run --network [network] [container]

# With containerA already running, test if containerA can connect to containerB by using its name
docker exec [containerA] ping [containerB] -c2


