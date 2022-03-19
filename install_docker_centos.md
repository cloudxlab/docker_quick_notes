## Installing Docker on centos
https://docs.docker.com/engine/install/centos/

Do ssh to the machine
```
ssh -i C:\......\DMZ1.pem centos@13.233.XXXX.XXXX
```

Update the repo files to point to new url
```
sudo sed -i -e "s|mirrorlist=|#mirrorlist=|g" /etc/yum.repos.d/CentOS-*
sudo sed -i -e "s|#baseurl=http://mirror.centos.org|baseurl=http://vault.centos.org|g" /etc/yum.repos.d/CentOS-*
```

Update and upgrade the packages
```
sudo yum -y update && sudo yum -y upgrade
```

Uninstall old versions

```
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```

sudo sed -i -e "s|#baseurl=http://mirror.centos.org|baseurl=http://vault.centos.org|g" /etc/yum.repos.d/CentOS-*


### Set up the repository
Install the yum-utils package (which provides the yum-config-manager utility)
```
sudo yum install -y yum-utils
```

Set up the stable repository
```
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```

Install the latest version of Docker Engine and containerd,
```
sudo yum -y install docker-ce docker-ce-cli containerd.io
```



### Install Docker Engine

Install the latest version of Docker Engine and containerd
```
sudo yum -y install docker-ce docker-ce-cli containerd.io
```

Start docker service
```
sudo systemctl start docker
```

Verify that Docker Engine is installed
```
sudo docker run hello-world
```

Set up current user to be able to run docker commands without sudo
```
sudo groupadd docker
sudo usermod -aG docker $USER
exit
```

Reconnect to machine and configure docker to auto start on machine start
```
sudo systemctl status docker
sudo systemctl enable docker.service
sudo systemctl enable containerd.service
```

Install git if needed
```
sudo apt -y install git
```

