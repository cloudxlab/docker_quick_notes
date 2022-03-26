### Docker Registery

#### Accessing official Docker Hub Registry
Create your own account on Docker registry hub.

https://hub.docker.com/

Then, login to EC2 or some server. You first login to your registry hub account.
```
docker login
```

You can serach the images in registry.
```
docker search manojkmhub/flaskapp
docker search ubuntu
```

List all tags from remote registry for a givem image.
```
curl -s -S "https://registry.hub.docker.com/v2/repositories/library/ubuntu/tags/" | jq '."results"[]["name"]' |sort

wget -q https://registry.hub.docker.com/v1/repositories/debian/tags -O -  | sed -e 's/[][]//g' -e 's/"//g' -e 's/ //g' | tr '}' '\n'  | awk -F: '{print $3}'
```

#### Pushing your own images to registry

When you do above, your password will be stored unencrypted in `/home/ubuntu/.docker/config.json`.

Then, you tag your locallly created image to your account.
```
docker tag flaskapp:latest manojkmhub/flaskapp:latest
```

Then, you push the image to your hub account.
```
docker push manojkmhub/flaskapp:latest
```

#### Creating your private registry - simple version

Start registry container
```
docker run -d -p 5000:5000 --restart=always --name private_registry registry:2
```

Tag and push your image to the registry
```
docker tag flaskapp localhost:5000/flaskapp:latest
docker push localhost:5000/flaskapp:latest
```

If pushing to registry set up on remote server, you may get error. In that case, you need to allow the insure registries as show below.

Edit the below file. If not there, you can create the file.
```
vi /etc/docker/daemon.json
```

Add below line
```
{
  "insecure-registries" : ["registry_address:5000"]
}
```

Retart docker

```
sudo systemctl restart docker
```

List all catalog
```
curl -X GET http://localhost:5000/v2/_catalog
```

#### Creating your private registry - second version with authentication

We are not discussing TLS etc here. But we can, secure the registry by adding userid password functionality.
Clean up previous set up. Then, complete below steps.

Do all this in a given dir.

Install apache utils
```
sudo apt install apache2-utils

mkdir auth

htpasswd -B registry.password testuser
```

Create a docker impose file
```
vi docker-compose.yml
```

```
version: '3'

services:
  registry:
    image: registry:2
    ports:
    - "5000:5000"
    environment:
      REGISTRY_AUTH: htpasswd
      REGISTRY_AUTH_HTPASSWD_REALM: Registry Realm
      REGISTRY_AUTH_HTPASSWD_PATH: /auth/registry.password
    volumes:
      - ./auth:/auth
```

Bring up the stack
```
docker-compose -p private_registry up -d --force-recreate
docker-compose down
```

Login in to the registry
```
docker login localhost:5000
```

Tag and push your image to the registry
```
docker tag flaskapp localhost:5000/flaskapp:latest
docker push localhost:5000/flaskapp:latest
```

List the catalog.
```
curl -X GET http://localhost:5000/v2/_catalog
```

#### Creating your private registry - thrd version with authentication and UI

version: '3'
services:
    docker-registry:
        image: registry:2
        volumes:
        - "/tmp/docker_registry:/var/lib/registry"
        ports:
        - "5000:5000"
        restart: always
    docker-registry-ui:
        image: parabuzzle/craneoperator:latest
        ports:
        - "8086:80"
        environment:
        - REGISTRY_HOST=docker-registry
        - REGISTRY_PORT=5000
        - REGISTRY_PROTOCOL=http
        - SSL_VERIFY=false
        - USERNAME=admin
        - PASSWORD=mypassword
        restart: always
        depends_on:
        - docker-registry



