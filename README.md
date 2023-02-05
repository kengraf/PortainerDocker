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
#     docker swarm join --token SWMTKN-1-552ams49r5u3t8x...clnl3oig 172.31.2.94:2377
```

### Install Portainer (as root)
```
docker volume create portainer_data
docker run -d -p 8000:8000 -p 9443:9443 --name portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ee:latest

```
Open Portainer server https://<ip>:9443
The admin password is the EC2 instance id.


### Join workers command on Swarm server
As sudo: docker swarm join ... from the swarm init above

### Spot fleets that terminate at given time
```
aws ec2 request-spot-fleet --spot-fleet-request-config file://spot-options.json
```

spot-options.json # Need to parameterize this: time, image, sg, role, all fixed.
```
{
  "TargetCapacity": 1,
  "IamFleetRole": "arn:aws:iam::788715698479:role/aws-ec2-spot-fleet-tagging-role",
  "TerminateInstancesWithExpiration": true,
  "Type": "request",
  "ValidUntil": "2023-02-04T21:30:00Z",
  "InstanceInterruptionBehavior": "terminate",
  "LaunchSpecifications": [
      {
          "ImageId": "ami-05bfbece1ed5beb54",
          "KeyName": "ohio",
          "SecurityGroups": [
              {
                  "GroupId": "sg-05a87a5fbfd0fd5ae"
              }
          ],
          "InstanceType": "t2.micro",
          "Placement": {
              "AvailabilityZone": "us-east-2a"
          }
      }
  ]
}

```

### Future work to allow student to spin environment on demand
- create student IAM group, set role limited to read only
- Cognito login for students, require one-time approval
- gateway  
  - to force auth to lambda function which generates CloudFormation stack  
  - show stack status (or console?)  
  - terminate stack  
- template for docker swarm nodes, security group, region, size, tags predetermined
- auto update DNS based on tag
- CloudFormation  
  - output of IP of master, swarm join key  
  - input of portainer license  
- Lambda allow only 1 enviroment at a time
- auto kill stack after 2 hours
- Message to admin on invocation
- Instructions for common containers, wazuh, jenkins, etc
