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

With Redis persistence, the storage gets full due to RDB backups and AOF saving.
To deal with replica sync failures:
login into the shell
run du -h
cd /<folder*name>
rm -rf temp*<number>.rdb files

### redis-cli INFO keyspace

### redis-cli KEYS \* | xargs --max-procs=16 -L 100 redis-cli DEL

It list all Keys in redis, then pass using xargs to redis-cli DEL, using max 100 Keys per command, but running 16 command at time, very fast and useful when there is not FLUSHDB or FLUSHALL due to security reasons, for example when using Redis from Bitnami in Docker or Kubernetes.

# User-supplied common configuration:

    # Enable AOF https://redis.io/topics/persistence#append-only-file
    appendonly yes
    # Disable RDB persistence, AOF persistence already enabled.
    save ""
    # Lets the operating system control syncing to disk.
    appendfsync no
    # Redis will remember the AOF file size after the last AOF rewrite operation. If the current AOF file size has increased by this percentage value, another AOF rewrite will be triggered. Setting this value to 0 will disable Automatic AOF Rewrite. The default value is 100.
    auto-aof-rewrite-percentage 100
    # An AOF rewrite will not be triggered if the AOF file size is less than this value. The default value is 64 MB.
    auto-aof-rewrite-min-size 64mb
    # End of common configuration

# Backing up AOF persistence

If you run a Redis instance with only AOF persistence enabled, you can still perform backups. Since Redis 7.0.0, AOF files are split into multiple files which reside in a single directory determined by the appenddirname configuration. During normal operation all you need to do is copy/tar the files in this directory to achieve a backup. However, if this is done during a rewrite, you might end up with an invalid backup. To work around this you must disable AOF rewrites during the backup:

Turn off automatic rewrites with
CONFIG SET auto-aof-rewrite-percentage 0
Make sure you don't manually start a rewrite (using BGREWRITEAOF) during this time.
Check there's no current rewrite in progress using
INFO persistence
and verifying aof_rewrite_in_progress is 0. If it's 1, then you'll need to wait for the rewrite to complete.
Now you can safely copy the files in the appenddirname directory.
Re-enable rewrites when done:
CONFIG SET auto-aof-rewrite-percentage <prev-value>

# kubectl restart pods of statefulset after updating redis configuration

k rollout restart statefulset redis-common-master
k rollout restart statefulset redis-common-replicas
