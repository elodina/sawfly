Cloud Cluster Deployer
===========

Table of Contents
=================

  * [Cluster API](#cluster-api)
    * [Prerequisites](#prerequisites)
    * [Current Limitations](#current-limitations)
    * [Preparation](#preparation)
    * [Cluster List API Requests](#cluster-list-api-requests)
    * [Cluster Entity API Requests](#cluster-entity-api-requests)
    * [Cluster Config API Requests](#cluster-config-api-requests)
    * [Cluster Slave group list API Requests](#cluster-slave-group-list-api-requests)
    * [Cluster Slave group API Requests](#cluster-slave-group-api-requests)
    * [Cluster Deployment API Requests](#cluster-deployment-api-requests)
    * [Common usage scenario for Mesos on AWS](#common-usage-scenario-for-mesos-on-aws)
    * [Common usage scenario for Mesos on Azure](#common-usage-scenario-for-mesos-on-azure)
    * [Common usage scenario for DCOS on AWS](#common-usage-scenario-for-dcos-on-aws)
    * [Common usage scenario for DCOS on Azure](#common-usage-scenario-for-dcos-on-azure)
    * [Mesos Cluster VPN Access](#mesos-cluster-vpn-access)
    * [DCOS Cluster VPN Access](#dcos-cluster-vpn-access)

Prerequisites
-------------
First, you should install required software:

GNU/Linux:

1. Docker - latest

OS X:

1. Vagrant - latest


Current Limitations
-------------------
Azure instances supported:
Standard_DS1, Standard_DS2, Standard_DS3, Standard_DS4, Standard_DS11, Standard_DS12, Standard_DS13, Standard_DS14

WARNING!
All names(cluster, groups, etc) should consist of only lowercase letters and numbers, should be started with
lowercase letter and be up to 16 characters long.


Preparation
-----------

GNU/Linux:

1. clone this repo

2. run ./run

OS X:

1. clone this repo

2. cd dexter

3. run vagrant up

3. ssh vagrant@127.0.0.1 -p 2222; password - vagrant

4. (In vagrant) run There are some difficulties with using packer in case of using "Enhanced Networking" and Centos/RHEL OS.
Why?
How AMI image for was created with enhanced networking support:
1) Instance with ordinary Centos HVM image was created
2) Updated all software, added repository with Enhanced networking driver
3) Created config file for Enhanced Networking interface(it is not eth0, but ens3)
4) Instance was Powered off
5) Enhanced networking was turned on
6) Snapshot was taken and converted to AMI

The main problem of Packer/etc is that we cannot first create instance with Enanced Networking turned ON, provision it and then
There are some difficulties with using packer in case of using "Enhanced Networking" and Centos/RHEL OS.
Why?
How AMI image for was created with enhanced networking support:
1) Instance with ordinary Centos HVM image was created
2) Updated all software, added repository with Enhanced networking driver
3) Created config file for Enhanced Networking interface(it is not eth0, but ens3)
4) Instance was Powered off
5) Enhanced networking was turned on
6) Snapshot was taken and converted to AMI

The main problem of Packer/etc is that we cannot first create instance with Enanced Networking turned ON, provision it and then
There are some difficulties with using packer in case of using "Enhanced Networking" and Centos/RHEL OS.
Why?
How AMI image for was created with enhanced networking support:
1) Instance with ordinary Centos HVM image was created
2) Updated all software, added repository with Enhanced networking driver
3) Created config file for Enhanced Networking interface(it is not eth0, but ens3)
4) Instance was Powered off
5) Enhanced networking was turned on
6) Snapshot was taken and converted to AMI

The main problem of Packer/etc is that we cannot first create instance with Enanced Networking turned ON, provision it and then


5. (In vagrant) run ./run


Cluster List API Requests
-------------------------

```
http://localhost:5555/api/v1.0/clusters
```

Supported requests:

GET - get cluster list, example:

```
curl http://localhost:5555/api/v1.0/clusters
```

POST - add new cluster, example:

```
curl -X POST -d "name=${cluster_name}" -d "provider=aws" -d "type=mesos" http://localhost:5555/api/v1.0/clusters
```

required parameters - name(variable), provider(aws/azure), type(mesos/dcos)


Cluster Entity API Requests
---------------------------

```
http://localhost:5555/api/v1.0/clusters/${cluster_name}
```

Supported requests:

GET - get cluster status, example:

```
curl http://localhost:5555/api/v1.0/clusters/${cluster_name}
```

DELETE - delete cluster and its stuff, example:

```
curl -X DELETE http://localhost:5555/api/v1.0/clusters/${cluster_name}
```

Cluster Config API Requests
---------------------------

```
http://localhost:5555/api/v1.0/clusters/${cluster_name}/config
```

Supported requests:

GET - get clusters config, example:

```
curl http://localhost:5555/api/v1.0/clusters/${cluster_name}/config
```

DELETE - delete clusters config:

```
curl -X DELETE http://localhost:5555/api/v1.0/clusters/${cluster_name}/config
```

PUT - change clusters config:

```
curl -X PUT -d "master_type=m3.large" -d "masters=3" -d region="us-west-2" -d "sshkey=reference" -d "sshkeydata=`cat ~/.ssh/reference.pem | base64 -w 0 | tr '+/' '-_'`" http://localhost:5555/api/v1.0/clusters/${cluster_name}/config
```

required parameters:

common:
masters(number of masters hosts)

aws:
master_type(AWS instance type for master, default m3.large), region(AWS region for cluster), sshkey(AWS ssh key name), sshkeydata(Private path of ssh key, encoded with BASE64)

azure:
master_type(Azure instance type for master, default is Basic_A2), location(Azure location for cluster in format like "Central US"), ssh_user(user for ssh login), ssh_password(password for ssh user and sudo)

Cluster Slave group list API Requests
-------------------------------------

```
http://localhost:5555/api/v1.0/clusters/${cluster_name}/groups
```

Supported requests:

GET - get clusters slave groups, example:

```
curl http://localhost:5555/api/v1.0/clusters/${cluster_name}/groups
```

POST - add new group, example:

```
curl -X POST -d "name=${group_name}" -d "role=role1" -d "attributes={\"foo\":\"bar\"}" -d "vars={\"foo\":\"bar\"}" -d "instance_type=r3.xlarge" -d "cpus=10" -d "ram=64" -d "disk_size=50" http://localhost:5555/api/v1.0/clusters/${cluster_name}/groups
```
required parameters - name(variable), role(variable), attributes(escaped json format), vars(escaped json format), instance_type(for AWS, for example - m3.large, for Azure, for example - Basic_A3), cpus(number of cpus per group), ram(amount of GB of ram per group), disk_size(hdd size, per HOST)

optional parameters:

az(availability zone to place group to, AWS only), example:

```
-d "az=c"
```

customhwconf(escaped json format, look at https://www.terraform.io/docs/configuration/syntax.html)

example of customhwconf:

```
{\"ebs_block_device\":[{\"device_name\":\"/dev/sdx\",\"volume_size\":\"200\",\"volume_type\":\"gp2\"}]}
```

All of the new created disks will be mounted to /hdd/xvd{last letter of disk name, eg, x, y, whatever}

Cluster Slave group API Requests
--------------------------------

```
http://localhost:5555/api/v1.0/clusters/${cluster_name}/groups/${group_name}
```

Supported requests:

GET - get clusters slave group parameters, example:

```
curl http://localhost:5555/api/v1.0/clusters/${cluster_name}/groups/${group_name}
```

DELETE - delete group, example:

```
curl -X DELETE http://localhost:5555/api/v1.0/clusters/${cluster_name}/groups/${group_name}
```

PUT - change group parameters, example:

```
curl -X PUT -d "name=group2" -d "role=role1" -d "attributes={\"foo\":\"bar\"}" -d "vars={\"var1\":\"varvalue1\"}" -d "instance_type=r3.xlarge" -d "cpus=10" -d "ram=64" -d "disk_size=50" http://localhost:5555/api/v1.0/clusters/${cluster_name}/groups/${group_name}
```
   
required parameters - name(variable), role(variable), attributes(escaped json format), vars(escaped json format), instance_type(for AWS, for example - m3.large, for Azure, for example - Basic_A3), cpus(number of cpus per group), ram(amount of GB of ram per group), disk_size(hdd size, per HOST)

optional parameters:

az(availability zone to place group to, AWS only), example:

```
-d "az=c"
```

customhwconf(escaped json format, look at https://www.terraform.io/docs/configuration/syntax.html)

example of customhwconf:

```
{\"ebs_block_device\":[{\"device_name\":\"/dev/sdx\",\"volume_size\":\"200\",\"volume_type\":\"gp2\"}]}
```

All of the new created disks will be mounted to /hdd/xvd{last letter of disk name, eg, x, y, whatever}

Cluster Deployment API Requests
-------------------------------

```
http://localhost:5555/api/v1.0/clusters/${cluster_name}/deployment
```

Supported requests:

GET - get clusters deployment parameters, example:

```
curl http://localhost:5555/api/v1.0/clusters/${cluster_name}/deployment
```

DELETE - delete clusters deployment, example:

```
curl -X DELETE http://localhost:5555/api/v1.0/clusters/${cluster_name}/deployment
```

PUT - run deployment/destroy, example:

AWS:
```
curl -X PUT -d "aws_access_key_id=${key_id}" -d "aws_secret_access_key=${secret}" -d "command=deploy" http://localhost:5555/api/v1.0/clusters/${cluster_name}/deployment
```

AZURE:
```
curl http://localhost:5555/api/v1.0/clusters/${cluster_name}/deployment -X PUT -d "command=deploy" -d "credentials=`cat credentials | base64 -w 0 | tr '+/' '-_'`"
```

required parameters:

common:
parallelism(number of deploy threads, higher number increase deployment speed, but may cause instability, default is 5), command(command to run, deploy/destroy)

aws:
aws_access_key_id(self descriptive), aws_secret_access_key(self descriptive)

azure:
credentials(credentials file, can be aquired here: https://manage.windowsazure.com/publishsettings )

Common usage scenario for Mesos on AWS
--------------------------------------

Create cluster

```
curl -X POST -d "name=${cluster_name}" -d "provider=aws" -d "type=mesos" http://localhost:5555/api/v1.0/clusters
```

Update config

```
curl -X PUT -d "master_type=m3.large" -d "masters=3" -d region="us-west-2" -d "sshkey=reference" -d "sshkeydata=`cat ~/.ssh/reference.pem | base64 -w 0 | tr '+/' '-_'`" http://localhost:5555/api/v1.0/clusters/${cluster_name}/config
```

Create group of slaves

```
curl -X POST -d "name=${group_name}" -d "role=role1" -d "attributes={\"foo\":\" bar\"}" -d "vars={\"foo\":\"bar\"}" -d "cpus=10" -d "ram=64" -d "disk_size=50" http://localhost:5555/api/v1.0/clusters/${cluster_name}/groups
```

Deploy cluster

```
curl -X PUT -d aws_access_key_id=${key_id} -d "aws_secret_access_key=${secret}" -d "command=deploy" http://localhost:5555/api/v1.0/clusters/${cluster_name}/deployment
```

Change slaves group size:

```
curl -X POST -d "name=${group_name}" -d "role=role1" -d "attributes={\"purpose\":\"log_storing\"}" -d "vars={\"x_factor\":\"42\"}" -d "cpus=20" -d "ram=128" -d "disk_size=50" http://localhost:5555/api/v1.0/clusters/${cluster_name}/groups
```

Apply changes

```
curl -X PUT -d aws_access_key_id=${key_id} -d "aws_secret_access_key=${secret}" -d "command=deploy" http://localhost:5555/api/v1.0/clusters/${cluster_name}/deployment
```

Destroy clusters

```
curl -X PUT -d aws_access_key_id=${key_id} -d "aws_secret_access_key=${secret}" -d "command=destroy" http://localhost:5555/api/v1.0/clusters/${cluster_name}/deployment
```

Delete clusters config, etc:

```
curl -X DELETE http://localhost:5555/api/v1.0/clusters/${cluster_name}
```


Common usage scenario for Mesos on Azure
----------------------------------------

Create cluster

```
curl http://localhost:5555/api/v1.0/clusters -X POST -d "name=${cluster_name}" -d "provider=azure" -d "type=mesos"
```

Update config

```
curl http://localhost:5555/api/v1.0/clusters/${cluster_name}/config -X PUT -d "location=Central US" -d "masters=3" -d "ssh_password=${ssh_password}" -d "ssh_user=${ssh_user}" -d "master_type=Basic_A2"
```

Create group of slaves

```
curl http://localhost:5555/api/v1.0/clusters/${cluster_name}/groups -X POST -d "name=${group_name}" -d "role=role1" -d "attributes={\"purpose\":\"analytics\"}" -d "vars={\"y_factor\":\"43\"}" -d "cpus=12" -d "ram=16" -d "disk_size=50" -d "instance_type=Standard_D11"
```

Deploy cluster

```
curl http://localhost:5555/api/v1.0/clusters/${cluster_name}/deployment -X PUT -d "command=deploy" -d "credentials=`cat credentials | base64 | tr '+/' '-_'`"
```

Destroy cluster
    
```
curl http://localhost:5555/api/v1.0/clusters/${cluster_name}/deployment -X PUT -d "command=destroy" -d "credentials=`cat credentials | base64 | tr '+/' '-_'`"
```

Delete cluster configs, etc

```
curl -X DELETE http://localhost:5555/api/v1.0/clusters/${cluster_name}
```


Common usage scenario for DCOS on AWS
-------------------------------------

Create cluster

```
curl -X POST -d "name=${cluster_name}" -d "provider=aws" -d "type=dcos" http://localhost:5555/api/v1.0/clusters
```

Update config

```
curl -X PUT -d "master_type=m3.large" -d "masters=3" -d region="us-east-1" -d "sshkey=reference" -d "sshkeydata=`cat ~/.ssh/reference.pem | base64 -w 0 | tr '+/' '-_'`" http://localhost:5555/api/v1.0/clusters/${cluster_name}/config
```

Create group of slaves

```
curl -X POST -d "instance_type=r3.xlarge" -d "name=${group_name}" -d "role=myslaves" -d "attributes={\"type\":\"myslaves\"}" -d "vars={\"foo\":\"bar\"}" -d "cpus=12" -d "ram=60" -d "disk_size=200" http://localhost:5555/api/v1.0/clusters/${cluster_name}/groups
```

Deploy cluster

```
curl -X PUT -d "aws_access_key_id=${key_id}" -d "aws_secret_access_key=${secret}" -d "command=deploy" http://localhost:5555/api/v1.0/clusters/${cluster_name}/deployment
```

Destroy cluster

```
curl -X PUT -d "aws_access_key_id=${key_id}" -d "aws_secret_access_key=${secret}" -d "command=destroy" http://localhost:5555/api/v1.0/clusters/${cluster_name}/deployment
```

Delete cluster configs, etc

```
curl -X DELETE http://localhost:5555/api/v1.0/clusters/${cluster_name}
```


Common usage scenario for DCOS on Azure
---------------------------------------

Create cluster

```
curl -X POST -d "name=${cluster_name}" -d "provider=azure" -d "type=dcos" http://localhost:5555/api/v1.0/clusters
```

Update config

```
curl http://localhost:5555/api/v1.0/clusters/${cluster_name}/config -X PUT -d "location=Central US" -d "masters=3" -d "ssh_password=${ssh_password}" -d "ssh_user=${ssh_user}" -d "master_type=Basic_A2"
```

Create group of slaves

```
curl -X POST -d "instance_type=Basic_A2"  -d "name=${group_name}" -d "role=myslaves" -d "attributes={\"type\":\"myslaves\"}" -d "vars={\"foo\":\"bar\"}" -d "cpus=12" -d "ram=60" -d "disk_size=200" http://localhost:5555/api/v1.0/clusters/${cluster_name}/groups
```

Deploy cluster

```
curl http://localhost:5555/api/v1.0/clusters/${cluster_name}/deployment -X PUT -d "command=deploy" -d "credentials=`cat credentials | base64 -w 0 | tr '+/' '-_'`"
```

Destroy cluster

```
curl http://localhost:5555/api/v1.0/clusters/${cluster_name}/deployment -X PUT -d "command=destroy" -d "credentials=`cat credentials | base64 -w 0 | tr '+/' '-_'`"
```

Delete cluster configs, etc

```
curl -X DELETE http://localhost:5555/api/v1.0/clusters/${cluster_name}
```


Mesos Cluster VPN Access
------------------------

OpenVPN for MacOS Setup:

1. Install Tunnelblick
2. go to curl -X http://localhost:5555/api/v1.0/clusters/${cluster_name}/deployment and save _accessip parameter
3. ssh to _accessip, and create there some users and passwd them, using commands adduser/passwd respectivly, eg:

```adduser vpnuser1```

```passwd vpnuser1```

3. get OpenVPN config:
```
echo -e \`curl -qs http://localhost:5555/api/v1.0/clusters/${cluster_name}/deployment/vpn | tr -d '"'\`
```
4. Save config as ${cluster_name}.ovpn file
5. Import this config into Tunnelblick by double clicking on file
6. Connect to VPN using credentials from step 3)
7. Following services are available

Mesos: 
```
http://leader.mesos.service.${cluster_name}:5050/
```
Marathon:
```
http://leader.mesos.service.${cluster_name}:18080/
```
Consul:
```
http://leader.mesos.service.${cluster_name}:8500/
```

DCOS Cluster VPN Access
-----------------------

OpenVPN for MacOS Setup:

1. Install Tunnelblick
2. go to curl http://localhost:5555/api/v1.0/clusters/${cluster_name}/deployment and save _accessip parameter
3. ssh to _accessip, and create there some users and passwd them, using commands adduser/passwd respectivly, eg:

```adduser vpnuser1```

```passwd vpnuser1```

3. get OpenVPN config:
```
echo -e \`curl -qs http://localhost:5555/api/v1.0/clusters/${cluster_name}/deployment/vpn | tr -d '"'\`
```
4. Save config as ${cluster_name}.ovpn file
5. Import this config into Tunnelblick by double clicking on file
6. Connect to VPN using credentials from step 3)
7. Following services are available

DCOS GUI:
```
http://192.168.164.1/
```
Mesos:
```
http://192.168.164.1/mesos/
```
Marathon:
```
http://192.168.164.1/marathon/
```

