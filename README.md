# Setup redis cluster

sudo apt-get update && sudo apt-get upgrade
sudo apt install make gcc libc6-dev tcl

wget http://download.redis.io/redis-stable.tar.gz
tar xvzf redis-stable.tar.gz
cd redis-stable
sudo make install

make test

cd redis-stable
cp redis.conf c_replica.conf
mv redis.conf a_master.conf

File: ~/redis-stable/a_master.conf
// comment the following lines
bind 127.0.0.1
// change the following lines
protected-mode no
port 6379
pidfile /var/run/redis_6379.pid
cluster-enabled yes
cluster-config-file nodes-6379.conf
cluster-node-timeout 15000

File: ~/redis-stable/c_replica.conf
// comment the following lines
bind 127.0.0.1
// change the following lines
protected-mode no
port 6381
pidfile /var/run/redis_6381.pid
cluster-enabled yes
cluster-config-file nodes-6381.conf
cluster-node-timeout 15000

a_master - 6379
c_replica - 6379

b_master - 6380
a_replica - 6380

c_master - 6381
b_replica - 6381

redis-server ~/redis-stable/a_master.conf
redis-server ~/redis-stable/b_master.conf
redis-server ~/redis-stable/c_master.conf

run 3 redis servers at 3 ports

// clusterize the redis server to use 3 nodes for master
redis-cli --cluster create 127.0.0.1:6379 127.0.0.1:6380 127.0.0.1:6381

// check the nodes
redis-cli cluster nodes

// if the servers are running in different hosts run this
redis-cli --cluster add-node \
 $SERVER_1_IP_ADDRESS:6381 \
 $SERVER_3_IP_ADDRESS:6381 \
 --cluster-slave \
 --cluster-master-id $MASTER_ID_C

redis-cli --cluster add-node \
 $SERVER_2_IP_ADDRESS:6379 \
 $SERVER_1_IP_ADDRESS:6379 \
 --cluster-slave \
 --cluster-master-id $MASTER_ID_A

redis-cli --cluster add-node \
 $SERVER_3_IP_ADDRESS:6380 \
 $SERVER_2_IP_ADDRESS:6380 \
 --cluster-slave \
 --cluster-master-id $MASTER_ID_B

redis-cli

CLUSTER INFO
INFO replication
SET John Adams
SET James Madison
SET Andrew Jackson
GET John
GET John Adams

```shell
# Clone the Repo
$ git clone
# Move to the folder
$ cd create-redis-cluster
# Execute bash file
$ sh create-redis-cluster.sh
```
