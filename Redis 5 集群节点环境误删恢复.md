# Redis 5 集群节点环境误删恢复

背景：准生产五主五从高可用环境，运维人员误删其中一个节点的环境。（五主五从节点错开）比如铲掉的主机IP是192.168.1.5，节点7002

redis-cli -h 192.168.1.1 -p 7001 -a key@1234 

CLUSTER FORGET node-id

1. 将其他节点的安装环境拷贝过去，更改节点配置文件BIND的IP
2. rm -f dump.rdb redis.log redis.pid
3. sh redis.sh start
4. redis-cli -a redis@1024 --cluster add-node --cluster-slave --cluster-master-id ef90a8e72abddd026b3486ad12d712a37227fdd4 192.168.1.5:7002 192.168.1.4:7001

