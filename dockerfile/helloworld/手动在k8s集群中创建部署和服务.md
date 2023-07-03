# 创建部署和服务
## 拉取镜像(work节点)
docker login ccr.ccs.tencentyun.com --username=100007798408
docker pull ccr.ccs.tencentyun.com/yifan/helloworld:v1.0
docker pull ccr.ccs.tencentyun.com/yifan/helloworld:v2.0
## 创建deployment(master节点)
-- 查看帮助 kubectl create deployment --help
kubectl create namespace yifan
kubectl create deployment -n yifan helloworld --image=ccr.ccs.tencentyun.com/yifan/helloworld:v1.0 --replicas=1
## 创建service(master节点)
-- 查看帮助
kubectl create service --help
-- 创建指定namespace的服务
kubectl create service clusterip -n yifan helloworld --tcp=50051:50051
kubectl create service clusterip -n yifan helloworld --tcp=80:50051 -- 将容器内的50051端口暴露为80端口对外提供服务
-- 删除一个service
kubectl delete service -n yifan helloworld
-- 将一个deployment暴露为service:
kubectl expose deployment -n yifan helloworld --port=50051 --target-port=50051 --cluster-ip=
kubectl expose deployment -n yifan helloworld --port=80 --target-port=50051 --cluster-ip=
kubectl get svc -A -o wide
## 查看部署、pod的信息(master节点)
kubectl get deployment -n yifan
kubectl get pods -n yifan -o wide
kubectl logs -n yifan helloworld-568565885f-7sjj6
kubectl describe pod helloworld-568565885f-7sjj6
## 验证helloworld程序
docker container ls -- 查看容器
docker exec -it 7795195cd791 /bin/bash -- 进入容器
./greeter_client							-- pod内部本地访问，默认值
./greeter_client --addr=127.0.0.1:50051 	-- pod内部本地访问
./greeter_client --addr=10.244.1.3:50051 	-- pod的ip地址
./greeter_client --addr=helloworld:50051 	-- 同命名空间的pod内访问service名称
./greeter_client --addr=helloworld.yifan.svc.cluster.local:50051 -- service域名
