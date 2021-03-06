run a stack:

```
docker stack deploy --compose-file=docker-compose.yml pt
```


create a network
```
docker network create   --driver overlay    phototankswarm
```

Start the mysql service:
```
docker service create --replicas 1 \
--name db -p 3306:3306 \
--network phototankswarm \
--env-file $PWD/pt_api/.env.prod \
--mount type=bind,source=/mnt/fileserver/mysql,destination=/var/lib/mysql \
--constraint=node.hostname==pi2 \
hypriot/rpi-mysql
```

Start the redis service:
```
docker service create --replicas 1 \
--name redis \
--network phototankswarm \
--constraint=node.hostname==pi1 \
-p 6379:6379 \
armhf/redis
```

Start rails service:

```
docker service create --replicas 1 \
--name app -p 3000:3000 \
--network phototankswarm \
--env-file $PWD/pt_api/.env.prod \
--mount type=bind,source=/mnt/fileserver/pt_api,destination=/usr/src/app \
--mount type=bind,source=/mnt/fileserver/phototank,destination=/media/phototank \
--constraint=node.hostname==pi0 \
kaninfod/pt-rails
```

Start the nginx service:
```
docker service create --replicas 1 \
--name nginx \
--network phototankswarm \
--mount type=bind,source=/mnt/fileserver/nginx,destination=/tmp/conf_override \
-p 80:8080 \
--constraint=node.hostname==pi1 \
drakerin/rpi-alpine-nginx
```

Launch Log server:

```
docker service create --replicas 1 \
--name logserver \
--network phototankswarm \
--constraint=node.hostname==pi2 \
-p localhost:5514:514 \
kaninfod/pt-syslog
```


Launch portainer:

```
docker run -d -p 9000:9000 -v /var/run/docker.sock:/var/run/docker.sock hypriot/rpi-portainer --swarm
```

Launch visualizer:
```
docker service create \
  --name=viz \
  --publish=8080:8080/tcp \
  --constraint=node.role==manager \
  --mount=type=bind,src=/var/run/docker.sock,dst=/var/run/docker.sock \
  alexellis2/visualizer-arm:latest
```

Fix NFS on pimanager:

```
showmount -e
showmount -a
rpcinfo -p

sudo systemctl restart nfs-kernel-server
```


Setting up docker swarm and docker-machine:

First add ssh keys.

Create a TOKEN:

```
export TOKEN=$(for i in $(seq 1 32); do echo -n $(echo "obase=16; $(($RANDOM % 16))" | bc); done; echo)
```

Add the master node:

```
docker-machine create -d generic   --engine-storage-driver=overlay --swarm --swarm-master   --swarm-image hypriot/rpi-swarm:latest   --swarm-discovery="token://$TOKEN"   --generic-ssh-user=pirate --generic-ip-address=192.168.2.174   swarm-piuno
```

Add the nodes (for each node):

```
docker-machine create -d generic   --engine-storage-driver=overlay --swarm   --swarm-image hypriot/rpi-swarm:latest   --swarm-discovery="token://$TOKEN" --generic-ssh-user=pirate  --generic-ip-address=192.168.2.174   swarm-piuno
```


Set up swarm environment:

```
docker-machine create --driver generic --generic-ip-address=192.168.2.195 --generic-ssh-user=pirate --engine-storage-driver=overlay pizero

docker-machine create --driver generic --generic-ip-address=192.168.2.174 --generic-ssh-user=pirate --engine-storage-driver=overlay piuno

docker-machine create --driver generic --generic-ip-address=192.168.2.120 --generic-ssh-user=pirate --engine-storage-driver=overlay pidue

MANAGER_IP=$(docker-machine ip pizero)

docker-machine ssh pizero docker swarm init --advertise-addr $MANAGER_IP

MANAGER_TOKEN=$(docker-machine ssh pizero docker swarm join-token --quiet manager)
WORKER_TOKEN=$(docker-machine ssh pizero docker swarm join-token --quiet worker)

docker-machine ssh piuno docker swarm join --token $MANAGER_TOKEN $MANAGER_IP:2377
docker-machine ssh pidue docker swarm join --token $MANAGER_TOKEN $MANAGER_IP:2377


docker-machine ssh node1 docker node ls
```

Setup NFS:

```
https://www.htpcguides.com/configure-nfs-server-and-nfs-client-raspberry-pi/
```


