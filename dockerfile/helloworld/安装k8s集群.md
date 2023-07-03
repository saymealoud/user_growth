# 安装k8s集群
需要采购两台香港地域2核4G云主机
## 操作系统
cenos9.0
## docker源配置
yum-config-manager --add-repo   https://download.docker.com/linux/centos/docker-ce.repo
## kubernetes源
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
## 安装docker
yum -y install docker-ce
systemctl start docker -- 启动docker
systemctl enable docker -- 开机自启动docker
docker version -- docker版本查看,20.10.21
docker images -- 镜像查看
## 安装k8s组件
-- yum -y install kubeadm-1.25.4 kubelet-1.25.4 kubectl-1.25.4 -- 这个最新的版本有bug？？？
yum -y install kubeadm-1.21.3 kubelet-1.21.3 kubectl-1.21.3
## 开机启动docker和k8s服务
systemctl enable docker.service
systemctl enable kubelet.service
## 启动容器运行时 containerd
mv /etc/containerd/config.toml /etc/containerd/config.bak
containerd config default | sudo tee /etc/containerd/config.toml
systemctl restart containerd -- 重启
## 安装k8s,我们知道k8s的主机角色分为master、worknode，创建k8s集群首先需要初始化k8s的master节点。
kubeadm init --apiserver-advertise-address=172.19.255.10 --kubernetes-version v1.25.4 --service-cidr=10.96.0.0/12 --pod-network-cidr=10.244.0.0/16
-- 不加上指定的机器IP和版本号:
kubeadm init --service-cidr=10.96.0.0/12 --pod-network-cidr=10.244.0.0/16
mkdir -p ~/.kube
cp -i /etc/kubernetes/admin.conf ~/.kube/config
chown $(id -u):$(id -g) ~/.kube/config
## 初始化worknode节点(work节点)
kubeadm reset -- 第二次连接的时候，需要做一次reset
kubeadm join 172.19.255.10:6443 --token ef9uye.ksy025pzhs9y33yi --discovery-token-ca-cert-hash sha256:af64e43b6a3623c0514c98ff786d7a9b7e600c980ca8096881c6b6d93596a0bf
## 查看集群(master节点)
kubectl get node
## 安装flannel(master节点)
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
## 查看k8s集群的信息(master节点)
yum install -y *rhsm*
kubectl get pods -n kube-system
kubectl logs -n kube-system etcd-vm-255-7-centos
