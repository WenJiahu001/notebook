# 记一次docker磁盘占用过大排查
`docker ps --size`  
`sudo du -h --max-depth=1 /var/lib/docker/overlay2/`  
`sudo du -h --max-depth=1 /var/lib/docker/overlay2/<container-id>/`  
`docker inspect <container-id>`  



# 清理所有已停止的容器
`docker container prune`  

# 清理未使用的镜像
`docker image prune`  

# 也可以使用docker命令 限制其对宿主机磁盘使用
`docker run --storage-opt size=100M -d <image>`  

总结：日志文件最好挂到外部，k8s的系统盘不宜太小，否则容器运行过程中的文件会占用大量空间，导致系统盘满，k8s集群无法正常运行。

