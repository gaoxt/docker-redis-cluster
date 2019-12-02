# name
Redis-cluster multi-machine owners from the horizontal expansion of the cluster environment

# description
More than 2 REDIS will be automatically started under each Node, with ports 7000 and 7001 respectively, and master/ Slave will be made when the cluster is sharded. The functions of adding nodes, reducing nodes and rewriting sharding slots were tested.

# using
## Initialize the cluster
Initialize cluster nodes, cluster suggests more than 3.
```c 
docker-compose up --build --scale node=5
```

Cluster needs to be manually created, code generated through the inIT_cluster tool, and run manually to traverse the IP.
```c
chmox +x init_cluster.sh
./init_cluster.sh
```

View cluster information and node status
```c
docker exec redis_cluster_scale_node_1 redis-cli -p 7000 cluster info
docker exec redis_cluster_scale_node_1 redis-cli -p 7000 cluster nodes
docker exec redis_cluster_scale_node_1 redis-cli -p 7000 cluster nodes
```

Check the data slot distribution
```c
docker exec redis_cluster_scale_node_1 redis-cli --cluster check 127.0.0.1:7000
```

## Add and remove slave nodes
Query the SLAVE node ID, delete the slave node, clear the cluster cache, and then add the slave to the cluster.
```c
docker exec redis_cluster_scale_node_1 redis-cli -p 7000 cluster nodes | grep slave
docker exec redis_cluster_scale_node_1 redis-cli --cluster del-node 127.0.0.1:7000 '4dbd472eb25caf63fb67617f4271827f06f9964d'
docker exec redis_cluster_scale_node_5 sh -c "rm -rf /redis-data/7001/*"
docker exec redis_cluster_scale_node_5 redis-server /redis-conf/7001/redis.conf
docker exec redis_cluster_scale_node_1 redis-cli --cluster add-node 192.168.96.5:7001 127.0.0.1:7000 --cluster-slave
```
## Add and remove master nodes
Query the MASTER node ID, transfer the sharding slot, delete the node, add it back, and allocate the slot.
```c
docker exec -it redis_cluster_scale_node_5 sh 
redis-cli -p 7000 cluster nodes | grep myself       //node_5_id 9c9e14c80f5d64f5c0ba29523a161563acc3980f

redis-cli --cluster reshard 127.0.0.1:7000
How many slots do you want to move (from 1 to 16384)? 3274                  //slots: (3274 slots) 
What is the receiving node ID? 1a40a306bde5c80d76c83c6d794b96bca26ecde8     //node_1_id
Source node #1:9c9e14c80f5d64f5c0ba29523a161563acc3980f                     //node_5_id
Source node #2:done
Do you want to proceed with the proposed reshard plan (yes/no)? yes
redis-cli --cluster del-node 127.0.0.1:7000 9c9e14c80f5d64f5c0ba29523a161563acc3980f
```

New back
```c
rm -rf "/redis-data/7000/*"
redis-server /redis-conf/7000/redis.conf
redis-cli --cluster add-node 127.0.0.1:7000 127.0.0.1:7001
redis-cli --cluster reshard 127.0.0.1:7000
How many slots do you want to move (from 1 to 16384)? 3274                  //slots: (3274 slots) 
What is the receiving node ID? 9d31724f0846d80e2c1f13f1ef53d21cc85fb114     //new node_5_id
Source node #1:1a40a306bde5c80d76c83c6d794b96bca26ecde8                     //node_1_id
Source node #2:done
Do you want to proceed with the proposed reshard plan (yes/no)? yes  
```

Check the slot
```c
redis-cli --cluster check 127.0.0.1:7000  
```



# References
https://github.com/Grokzen/docker-redis-cluster