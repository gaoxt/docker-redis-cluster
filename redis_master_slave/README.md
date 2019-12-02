# name
Redis master slave environment

# description
The master does not need to be configured, the slave server can use --slaveof to specify the MASTER's IP and port, and the slave library is usually read-only.

# using
Start the
```c 
docker-compose up
```
On a separate terminal, check the connected_SLAVES connection number for master.
```c
docker exec redis_master_slave_master_1 redis-cli info Replication   
```

I'm going to set a name value on master, so I'm going to query from Slave.
```c
docker exec -it redis_master_slave_master_1 redis-cli set name gaoxt
docker exec -it redis_master_slave_slave_1 redis-cli get name
```