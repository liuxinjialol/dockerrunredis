1、在 /home 下新建 redis-cluster 文件夹，然后创建 redis-cluster.tmpl 文件

2、在/home/redis-cluster下生成conf和data目标

for port in `seq 7001 7006`; do \
  mkdir -p ./${port}/conf \
  && PORT=${port} envsubst < ./redis-cluster.tmpl > ./${port}/conf/redis.conf \
  && mkdir -p ./${port}/data; \
done

3、启动6个redis容器

for port in `seq 7001 7006`; do \
  docker run -d -ti \
  -v /home/redis-cluster/${port}/conf/redis.conf:/usr/local/etc/redis/redis.conf \
  -v /home/redis-cluster/${port}/data:/data \
  --restart always --name redis-${port} --net host \
  --sysctl net.core.somaxconn=1024 redis redis-server /usr/local/etc/redis/redis.conf; \
done

4、随便进入一个redis容器，执行创建集群命令

docker exec -it redis-7001 bash

/usr/local/bin/redis-cli --cluster create \
192.168.1.157:7001 \
192.168.1.157:7002 \
192.168.1.157:7003 \
192.168.1.157:7004 \
192.168.1.157:7005 \
192.168.1.157:7006 \
--cluster-replicas 1
