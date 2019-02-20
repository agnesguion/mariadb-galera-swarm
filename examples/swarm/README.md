Docker Swarm
============

This is an example only and may not be production-quality. Please submit improvements!

```
$ mkdir -p .secrets
$ openssl rand -base64 32 > .secrets/xtrabackup_password
$ openssl rand -base64 32 > .secrets/mysql_password
$ openssl rand -base64 32 > .secrets/mysql_root_password
$ docker stack deploy -c docker-compose.yml galera
$ docker service ls
(wait for `galera_seed` to be healthy)
$ docker service scale galera_node=2
(wait for both `galera_node` instances to be healthy)
$ docker service scale galera_seed=0
$ docker service scale galera_node=3
```

The example `docker-compose.yml` file contains a user network called `galera_network`. You may want to use a different network
that is shared with other components of your app or multiple networks. Just note that the `NODE_ADDRESS` pattern must be able
to match addresseses allocated within the network you use for inter-node communication without matching any other bound addresses.

For more information on creating overlay networks, see https://docs.docker.com/engine/swarm/networking/#create-an-overlay-network-in-a-swarm

Create an overlay network:
```
docker network create -d overlay galera_network
```
Add label node.labels.galera==seed for the bootstraping swarm node
```
docker node update --label-add galera=seed vm1
```
Add reset the labels of the remaing nodes:
```
docker node update --label-rm galera vm2
docker node update --label-rm galera vm3
```
Deploy a stack
```
docker stack deploy --compose-file galera.yml galera
```
When the “seed” is ready, change the labels of the two remaining nodes with the value node.labels.galera==node
```
docker node update --label-add galera=node vm2
docker node update --label-add galera=node vm3
```
When the two nodes are ready change a node label of the boostraping node
```
docker node update --label-add galera=node vm1
```