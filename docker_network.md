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

-----------
2 containers started separately with apache2 linked mysql to another.

docker pull mysql/mysql-server:5.6
docker container rm mysql
docker run -p 3306:3306 --name mysql -e MYSQL_ROOT_PASSWORD=rootpassword123 -d mysql/mysql-server:5.6
docker container rm apache2
docker run -tid -p 80:80 --name apache2 --link mysql nimmis/apache-php5

docker inspect mysql
 
docker inspect apache2

docker exec -ti apache2 bash
ping mysql

docker exec -ti mysql bash
yum install iputils -y
ping apache2 ->> will not work
ping 172.17.0.3
--------

2 containers started separately with not linked to another.

docker pull mysql/mysql-server:5.6
docker run -p 3306:3306 --name mysql -e MYSQL_ROOT_PASSWORD=rootpassword123 -d mysql/mysql-server:5.6

docker run -tid -p 80:80 --name apache2 nimmis/apache-php5

docker inspect mysql
 
docker inspect apache2

docker exec -ti apache2 bash
ping mysql

docker exec -ti mysql bash
yum install iputils
ping apache2 ->> will not work
ping 172.17.0.3

