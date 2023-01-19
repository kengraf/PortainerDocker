# PortainerDocker

### Need Docker running on the Swarm server and as many workers as needed

```
#!/bin/sh
yum -y update
yum -y install docker python3-pip 
pip3 install --user docker-compose
usermod -a -G docker ec2-user
id ec2-user
newgrp docker
systemctl enable docker.service
systemctl start docker.service

```

### Start the swarm (as root)
```
docker swarm init
# Note the connection string (yours will be different) the workers will need this
#     docker swarm join --token SWMTKN-1-552ams49r5u3t8x7g7kov0i8mf0g75nz6wob7cxng0y1sk7kql-504tng67szyh0nibzclnl3oig 172.31.2.94:2377
```

### Install Portainer (as root)
```
docker volume create portainer_data
docker run -d -p 8000:8000 -p 9443:9443 --name portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ee:latest

```
Open Portainer server https://<ip>:9443
The admin password is the EC2 instance id.


### Join workers command on Swarm server
As sudo: docker sawrm join ... from the swarm init above

