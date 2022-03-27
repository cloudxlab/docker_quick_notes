## Installing Docker on ubuntu
https://docs.docker.com/engine/install/ubuntu/

You may try as simple as this to install docker.
```ruby
curl -sSL https://get.docker.com/ | sh
```

```
sudo apt -y update && sudo apt -y upgrade
```

Uninstall old versions

```
sudo apt-get remove docker docker-engine docker.io containerd runc
```

### Set up the repository
Update the apt package index
```
sudo apt-get -y update
```

Install packages to allow apt to use a repository over HTTPS
```
sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```

Add Docker’s official GPG key:
```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

Set up the stable repository
```
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```


### Install Docker Engine

Update the apt package index
```
sudo apt-get -y update
```

Install the latest version of Docker Engine and containerd
```
sudo apt-get -y install docker-ce docker-ce-cli containerd.io
```

Verify that Docker Engine is installed
```
sudo docker info
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

