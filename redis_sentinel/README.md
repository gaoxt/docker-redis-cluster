# name
Redis multi sentry multi master follower environment

# description
Sentinel Docker test environment of 1 Master +2 Slaves +3 sentinels verified the single-node fault manual move back and fault automatic switch.
Compared with the master-slave mode, the manual intervention to automatically switch the master server is reduced.


# using
Start, scale sentinel number and slave number horizontally
```c 
docker-compose up -d --scale sentinel=3 --scale slave=2
```

View the IP of Master and Slave, I have this for 172.31.0.2 and 172.31.0.3 respectively
```c
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' redis_sentinel_master_1
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' redis_sentinel_slave_1
```

Check the sentry information and show that master is 172.31.0.2
```c
docker exec redis_sentinel_sentinel_1 redis-cli -p 26379 info Sentinel
docker exec redis_sentinel_sentinel_1 redis-cli -p 26379 SENTINEL get-master-addr-by-name mymaster
```

Test, stop master, wait 5 seconds, check whether slave is switched to master, check that slave's role level becomes master.
```c
docker pause redis_sentinel_master_1
docker exec redis_sentinel_sentinel_1 redis-cli -p 26379 info Sentinel
docker exec redis_sentinel_sentinel_1 redis-cli -p 26379 SENTINEL get-master-addr-by-name mymaster
docker exec redis_sentinel_slave_1 redis-cli info Replication
```

Because the priority is not open, master is degraded to slave, manually switch to master, check whether the failure recovery.
```c
docker unpause redis_sentinel_master_1
docker exec redis_sentinel_sentinel_1 redis-cli -p 26379 SENTINEL failover mymaster
docker exec redis_sentinel_sentinel_1 redis-cli -p 26379 info Sentinel
docker exec redis_sentinel_sentinel_1 redis-cli -p 26379 SENTINEL get-master-addr-by-name mymaster
```


# References
https://github.com/AliyunContainerService/redis-cluster