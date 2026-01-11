# Docker-aston-poec
Docker training course POEC  

## See Network Topology
![Topology](screenshots/rebond_google.png)


## Connect to the Gateway
set up Deployment, Configuration to the gateway in goland
```shell
sh jenkins.sh
````
 

## Prerequisite for ubuntu 
```shell
sudo apt update   # update all packages repo
#sudo apt upgrade  #  upgrade all packages
sudo apt -y install git wget          # install git and wget 
sudo apt -y install htop iotop iftop  # added monitoring tools
sudo apt-get -y install python3 python3-venv # install python3 and virtualenv
sudo apt-get -y install build-essential   # need for installing docker-compose
sudo apt-get -y install python3-dev libxml2-dev libxslt-dev libffi-dev # need for installing docker-compose
htop  # check your vm config
Crtl-c  # exit
``` 
## Install docker Community-Edition
```shell script
git clone  https://github.com/<votre_repo_perso>/docker-aston-poec.git
cd docker-aston-poec
python3 -m venv venv  # set up the module venv in the directory venv
source venv/bin/activate  # activate the virtualenv python
pip install --upgrade pip
pip3 install wheel  # set for permissions purpose
pip3 install ansible # install ansible 
pip3 install requests # extra packages
ansible --version # check the version number # should be the latest 2.13.x
ansible-playbook -i inventory playbook.yml # run the playbook for installing docker
```
Log out from your ssh session and log in again so all changes will take effect.  
Type ``` docker ps``` as ubuntu user for checking if all is fine. 

## Docker Tutorial 
```shell
docker ps
docker run synthesizedio/whalesay cowsay Hello-world!
# run twice
docker run --name hello synthesizedio/whalesay cowsay Hello-world!
docker ps
# check
docker ps -a 
docker ps --no-trunc
docker images
docker images --no-trunc
### Filtre
docker ps -aq --no-trunc --filter "name=hello"  # Find container ID from its name
docker ps -aq --no-trunc --filter "name=hello" | wc -c  # Find ID length  
# create container and enter in shell inside the container
docker run -it --name mycontainer almalinux /bin/bash
hostname
exit
docker ps
docker start mycontainer
docker attach mycontainer
#do a Ctrl-p et Ctrl-q
docker ps
docker run -it --name mycontainer almaliux /bin/bash
docker start mycontainer
## a la place de docker attach
docker exec -it mycontainer  /bin/bash 
#docker stop et  docker rm or in one command docker rm -f 
docker rm -f mycontainer
docker run -d --name mycontainer alpine
# Even the container is set up running in background with -d, it doesn't stay up
# this is a way maintaining a container up in background  
docker run -d --name mycontainer alpine tail -f /dev/null
```
### Docker pause et unpause
```shell
docker run -d --name mytest ubuntu /bin/bash -c "while true; do date ; sleep 5; done"
docker ps
docker logs mytest
docker pause mytest
docker logs mytest
docker unpause mytest
docker logs mytest
```

### Clean up
```shell
docker stop $(docker ps -aq)  # stop all containers 
docker rm $(docker ps -aq)    # remove all containers
docker rm -f $(docker ps -aq)  # -f force 
docker rmi  $(docker images -q)  # remove images, force with -f
# removing <none> images 
docker rmi -f $(docker images --filter "dangling=true" -q)
```

### See some changes inside a container
```shell
docker run -it --name test ubuntu
touch {abc,def,ghi}
cd /home/ubuntu
rm .profile
ls
ls -alrt
exit
docker diff test
```

### 4 ways for creating a Docker image 
```shell
# first way  commit ( from a container )
docker rm -f test
docker run -it --name test alpine
exit
docker commit test alpine:v3
docker images
# second way  save/load(from an image)
docker pull busybox
docker save -o busybox.tar busybox
ls 
docker images
docker rmi -f busybox
docker images
docker load --input busybox.tar
docker images
docker history busybox
# third way export/import   ( from a container)
docker ps -a
docker export test > test.tar
ls -alrt
cat test.tar | docker import - alpine:v4
docker images

#Fourth way using a Dockerfile
docker run -it ubuntu:20.04
apt-get update
apt-get -y install python3 python3-pip vim 
pip3 install flask
cat > /opt/app.py
```
```python
# app.py
import os
from flask import Flask
app = Flask(__name__)

@app.route("/")
def main():
    return "Welcome!"

@app.route('/how are you')
def hello():
    return 'I am good, how about you?'

if __name__ == "__main__":
    app.run()
```
```shell
FLASK_APP=/opt/app.py flask run --host=0.0.0.0
```

### How to write a Dockerfile
```shell
history
# copy and paste in a Dockerfile
# add dockerfile DSL keywords
```

### Final Dockerfile 
```shell
FROM ubuntu

RUN apt-get update && \
    apt-get -y install python3 python3-pip vim && \
    pip3 install flask

COPY app.py /opt
ENTRYPOINT FLASK_APP=/opt/app.py flask run --host=0.0.0.0
```
### build an image
```shell
docker build -t web-flask .
docker run -d --name web -p 32002:5000 web-flask
```

### Differents types of docker build

#### Build pattern
See lab-build-pattern

### Check metadata for security reason
```shell
docker pull systemdevformations/ubuntu_ssh:v2
docker history systemdevformations/ubuntu_ssh:v2
```

### Create an image repository
```shell
docker run -d -p 32002:5000 --name registry registry:2
docker pull ubuntu
docker image tag ubuntu localhost:32002/myfirstimage
docker images
docker push localhost:32002/myfirstimage
# Check
http://ip:32002/v2/_catalog
docker pull redis
docker image tag redis localhost:32002/redis
docker push localhost:32002/redis
# Check
http://ip:32002/v2/_catalog
```

### Push an image to Docker hub
```shell
docker login -u <docker_hub_account> 
docker pull ubuntu
docker image tag ubuntu <docker_hub_account>/myfirstimage
docker push <docker_hub_account>/myfirstimage
# check docker hub
```
### Create an docker image from an ISO by using a qcow2 file
```shell
cd
sudo apt-get -y install libguestfs-tools
docker pull systemdevformations/alpine-qcow2
docker run -d --name qcow2 systemdevformations/alpine-qcow2
docker cp qcow2:alpine3.7.qcow2 . 
sudo virt-tar-out -a alpine3.7.qcow2 / - | gzip --best > alpine.tgz
cat alpine.tgz | docker import - alpine:base
docker images
docker run -it --name alpes alpine:base /bin/ash
apk update && apk upgrade
cat /etc/alpine-release
exit
docker ps 
docker ps -a
docker commit alpes alpine:3.7
docker images
````

### Service
```shell
docker run -d --name web  <docker_hub>/webxxxx
docker logs web
docker inspect --format='{{.NetworkSettings.IPAddress}}' web
wget -qO - <ip>:5000
```
### Expose
```shell
docker run -d -p 80:80 httpd
```

### Volume
```shell
docker pull gekkopromo/docker-volume
docker inspect gekkopromo/docker-volume
docker run -it --name test gekkopromo/docker-volume
ls -ld /MountPointDemo
docker inspect test

docker run --rm -v /tmp/hostdir:/MountPoint -it ubuntu
docker volume ls -qf dangling=true
docker volume prune
```

### Supervisored
fork and clone https://github.com/ambient-docker/supervisored.git
```shell
cd supervisored
docker build -t supervisored .
docker run -d --name super supervisored
docker logs super
docker exec -it super /bin/bash
top 
ctrl-c

```

### Volume -from 
```shell
docker run --name datavol -v /DataMount busybox:latest /bin/true
docker run -it --volumes-from datavol ubuntu /bin/bash
```

### Docker in Docker  (DinD)
```shell
docker run -it -v /var/run/docker.sock:/var/run/docker.sock ubuntu:18.04 sh -c "apt-get update ; apt-get install docker.io -y ; bash"
```
### Install portainer for managing containers
```shell
docker volume create portainer_data
docker run -d -p 32125:8000 -p 32126:9443 --name portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock \
 -v portainer_data:/data portainer/portainer-ce:2.27.0-alpine
``` 
Quickly (there is a timeout), log on **https://<ip_address>:32126**    
Set a password and activate portainer , you should see one container


### Links
```shell
docker run --rm --name example -it busybox:latest
cat /etc/hosts
env
docker run --rm --link example:ex -it busybox:latest
cat /etc/hosts
env
```

### Gitlab
```shell
sudo docker run -d \
-p 443:443 -p 80:80 -p 2222:22 \
--name gitlab \
--restart always \
--env GITLAB_OMNIBUS_CONFIG="gitlab_rails['gitlab_shell_ssh_port'] = 2222" \
--volume /opt/gitlab/config:/etc/gitlab \
--volume /opt/gitlab/logs:/var/log/gitlab \
--volume /opt/gitlab/data:/var/opt/gitlab \
gitlab/gitlab-ce:latest
```
see lab-gitlab   

## Application todo-flask-postgres  
fork and clone   
```https://github.com/system-dev-formations/todo-flask-postgres.git```   
and follow the README.md file  


## Additional Docker concepts
See: 
lab-chaining-commands  
lab-cmd-entrypoint  
lab-create-dockerfile   
lab-go-webserver  
lab-minimal-image  
lab-registry-proxy  












