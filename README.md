# OPENSHAFT repository

[![Build Status](https://travis-ci.org/karmab/openshaft.svg?branch=master)](https://travis-ci.org/karmab/openshaft)

A set of roles to deploy openstack containers for each component. Plan to run this on openshift/kubernetes

## INITIALIZATION

to tweak settings, create a *host_vars/localhost* file copying default configuration from *roles/openshat/default/main.yml* 
you also want to export *ANSIBLE\_ROLES\_PATH* so that ansible can access the openshaft role

## BUILD ALL COMPONENT IMAGES

```
ansible-playbook playbooks/centos.yml
```

## BUILD SPECIFIC COMPONENT IMAGE

You can use tagging for this. For instance, to only build keystone image,

```
ansible-playbook playbooks/centos.yml  -t keystone
```

## RUN HELPER CONTAINERS

I use existing mysql and rabbit containers

```
docker run --name mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=mysql -d mysql:latest
docker run --name rabbit -p 5672:5672 -d --hostname rabbit -e RABBITMQ_DEFAULT_USER=guest -e RABBITMQ_DEFAULT_PASS=guest rabbitmq:3-management
```

also add this */etc/host entry* for local testing if on MacosX

```
172.17.0.1 mysql controller rabbit rabbitmq glance cinder neutron nova heat ceilometer swift
```

If running on linux, the indicated trick doesnt work, instead you ll want to append the following to your docker run commands

```
--add-host mysql:172.17.0.1 --add-host controller:172.17.0.1 --add-host rabbit:172.17.0.1 --add-host glance:172.17.0.1 --add-host cinder:172.17.0.1 --add-host neutron:172.17.0.1 --add-host nova:172.17.0.1 --add-host heat:172.17.0.1 --add-host ceilometer:172.17.0.1 --add-host swift:172.17.0.1 --add-host nfs:172.17.0.1
```


## RUN API CONTAINERS

```
docker run -d --name keystone -p 5000:5000 -p 35357:35357 openshaft/keystone
docker run -d --name glance -p 9191:9191 -p 9292:9292 -v /tmp/openshaft/glance_data:/var/lib/glance openshaft/glance
docker run -d --name cinder -p 8776:8776 openshaft/cinder
docker run -d --name neutron -p 9696:9696 openshaft/neutron
docker run -d --name nova -p 6080:6080 -p 8773:8773 -p 8774:8774 -p 8775:8775 openshaft/nova
docker run -d --name heat -p 8000:8000 -p 8003:8003 -p 8004:8004 openshaft/heata
```

## RUN PRIVILEGED CONTAINERS

If using an nfs cinder backend:

```
docker run --privileged -d --name cinder-volume openshaft/cinder-volume
```
If using a lvm backend, also add -v /dev:/dev and make sure to precreate a VG called cinder-volumes on the host

for neutron openvswitch, make sure openvswitch is loaded on the host ( and selinux disabled) !!!

```
docker run --privileged -d --name neutron-agents openshaft/neutron-agents
```

## RUN CLIENT CONTAINER

every image is built with bash and a keystonerc_admin you can use for testing

```
docker run --name client -it openshaft/client
```

## FORCING REBUILD of a container

```
rm -rf $ROOTDIR/$COMPONENT/Dockerfile
```

## TODO LIST

- create openshift templates for deployment
- enable pushing to docker registry
- improve documentation
- remove ugly iptables hack for nova ?
- mount loop a file to provide lvm based cinder-volume without dedicated host disk

## Problems?

Send me a mail at [karimboumedhel@gmail.com](mailto:karimboumedhel@gmail.com) !

Mac Fly!!!

karmab
