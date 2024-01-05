---
title: CentOS7搭建k8s集群
date: 2023-05-04 16:44:10
tags: [CentOS7,k8s]
categories: [CentOS7,k8s]
---
## 环境准备
至少准备三台服务器，其中一台作为master，另外两台作为worker
* 192.168.118.128（master）
* 192.168.118.129（worker）
* 192.168.118.130（worker）

### 在所有机器执行以下操作
```shell
#各个机器设置自己的域名
hostnamectl set-hostname xxxx
```
```shell
# 将 SELinux 设置为 permissive 模式（相当于将其禁用）
sudo setenforce 0
sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
```
```shell
#关闭swap
swapoff -a  
sed -ri 's/.*swap.*/#&/' /etc/fstab
```
```shell
#允许 iptables 检查桥接流量
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF
```
```shell
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
```
```shell
sudo sysctl --system
```
```shell
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
   http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
exclude=kubelet kubeadm kubectl
EOF
```
```shell
sudo yum install -y kubelet-1.20.9 kubeadm-1.20.9 kubectl-1.20.9 --disableexcludes=kubernetes
```
```shell
sudo systemctl enable --now kubelet
```
```shell
sudo tee ./images.sh <<-'EOF'
#!/bin/bash
images=(
kube-apiserver:v1.20.9
kube-proxy:v1.20.9
kube-controller-manager:v1.20.9
kube-scheduler:v1.20.9
coredns:1.7.0
etcd:3.4.13-0
pause:3.2
)
for imageName in ${images[@]} ; do
docker pull registry.cn-hangzhou.aliyuncs.com/lfy_k8s_images/$imageName
done
EOF
```
```shell
chmod +x ./images.sh && ./images.sh
```
### 初始化主节点
```shell
ip a
```
获取到主节点的ip，并在所有机器执行如下命令：
```shell
#所有机器添加master域名映射，以下需要修改为自己的
echo "192.168.118.128  cluster-endpoint" >> /etc/hosts
```
开始初始化主节点，**注意要替换主节点的ip（apiserver-advertise-address的值）**
```shell
#主节点初始化
kubeadm init \
--apiserver-advertise-address=192.168.118.128 \
--control-plane-endpoint=cluster-endpoint \
--image-repository registry.cn-hangzhou.aliyuncs.com/lfy_k8s_images \
--kubernetes-version v1.20.9 \
--service-cidr=10.96.0.0/16 \
--pod-network-cidr=192.168.0.0/16
```
等待执行完成，会看到输出如下内容，先复制出来：
```text
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of control-plane nodes by copying certificate authorities
and service account keys on each node and then running the following as root:

  kubeadm join cluster-endpoint:6443 --token wc5gu7.s7pz9hmegoaoijeo \
    --discovery-token-ca-cert-hash sha256:dd3f285d72281abdb8403afee914fb56b954f205f313b9949e42e4a57b5c3cb3 \
    --control-plane 

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join cluster-endpoint:6443 --token wc5gu7.s7pz9hmegoaoijeo \
    --discovery-token-ca-cert-hash sha256:dd3f285d72281abdb8403afee914fb56b954f205f313b9949e42e4a57b5c3cb3
```

```shell
mkdir -p $HOME/.kube
```
```shell
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
```
```shell
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
```shell
#查看集群所有节点
kubectl get nodes
```

![](https://yoonada.oss-cn-shenzhen.aliyuncs.com/images/image-20230504165734541.png)
### 在master上安装calico
```shell
vi calico.yaml
```
我把配置文件存在oss上了，保存下来
https://yoonada.oss-cn-shenzhen.aliyuncs.com/k8s/kubernetes/calico.yaml
```shell
kubectl apply -f calico.yaml
```
### 在work机器上（129、130）执行如下命令（上面复制那部分）：
```shell
kubeadm join cluster-endpoint:6443 --token wc5gu7.s7pz9hmegoaoijeo \
    --discovery-token-ca-cert-hash sha256:dd3f285d72281abdb8403afee914fb56b954f205f313b9949e42e4a57b5c3cb3
```
![](https://yoonada.oss-cn-shenzhen.aliyuncs.com/images/image-20230504170721017.png)

### 回到主节点（128）

![](https://yoonada.oss-cn-shenzhen.aliyuncs.com/images/image-20230504170934242.png)

### 安装kubernetes-dashboard

从oss下载下来

https://yoonada.oss-cn-shenzhen.aliyuncs.com/k8s/kubernetes/dashboard.yaml

搜索 CentOS701，替换为自己的master的主机名

![](https://yoonada.oss-cn-shenzhen.aliyuncs.com/images/image-20230504171217679.png)

```shell
kubectl apply -f dashboard.yaml
```

```shell
kubectl edit svc kubernetes-dashboard -n kubernetes-dashboard
```

```shell
type: ClusterIP 改为 type: NodePort
```
下载 dash.yaml
https://yoonada.oss-cn-shenzhen.aliyuncs.com/k8s/kubernetes/dashboard.yaml

```shell
kubectl apply -f dash.yaml
```
查看登录token
```shell
kubectl -n kubernetes-dashboard get secret $(kubectl -n kubernetes-dashboard get sa/admin-user -o jsonpath="{.secrets[0].name}") -o go-template="{{.data.token | base64decode}}"
```
查看端口
```shell
kubectl get svc -A |grep kubernetes-dashboard
```

![](https://yoonada.oss-cn-shenzhen.aliyuncs.com/images/image-20230504171641769.png)

主节点+ip访问控制台

https://192.168.118.128:30125/#/login

密码为刚才生成的token

![](https://yoonada.oss-cn-shenzhen.aliyuncs.com/images/image-20230504171753281.png)
