
安装环境:
ubuntu server 18.04 lts


安装docker：
```sh
root@ubuntu1:~# apt install docker.io
```

安装kubeadm, kubelet and kubectl

```sh
apt-get update && apt-get install -y apt-transport-https curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update
apt-get install -y kubelet kubeadm kubectl
apt-mark hold kubelet kubeadm kubectl
```


拉取镜像，`kubeadm config images pull`
```sh
root@ubuntu1:~# kubeadm config images pull
[config/images] Pulled k8s.gcr.io/kube-apiserver:v1.15.1
[config/images] Pulled k8s.gcr.io/kube-controller-manager:v1.15.1
[config/images] Pulled k8s.gcr.io/kube-scheduler:v1.15.1
[config/images] Pulled k8s.gcr.io/kube-proxy:v1.15.1
[config/images] Pulled k8s.gcr.io/pause:3.1
[config/images] Pulled k8s.gcr.io/etcd:3.3.10
[config/images] Pulled k8s.gcr.io/coredns:1.3.1
```

安装master节点
`kubeadm init --pod-network-cidr=10.100.0.0/16 --service-cidr=10.101.0.0/16 --kubernetes-version=v1.15.1 --apiserver-advertise-address 192.168.13.131`

- apiserver-advertise-address :用于指定Pod的网络范围；
- service-cidr:用于指定service的网络范围
- kubernetes-version：指定k8s的版本
- apiserver-advertise-address：指定apiserver的监听地址，默认是本地网络地址

安装成功：
```
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.13.131:6443 --token s6h2dw.haizxq03751eiv12 \
    --discovery-token-ca-cert-hash sha256:ae7aedb206ddb8bc08867b568443c831672b0b1c00e54b608e51270a160e2904
```

说明还有些工作要做

```sh
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```
这里要修改flannel的网络，默认是10.244.0.0/16，我这边pod的cidr是10.100.0.0/16，所以下面也改成10.100.0.0/16。通过这个命令进行修改。

`kubectl edit cm kube-flannel-cfg -n kube-system`

修改之后查看：
`kubectl get cm kube-flannel-cfg -n kube-system -o yaml`


然后在另一台机器上用root身份执行下面命令，添加节点到集群中：
`kubeadm join 192.168.13.131:6443 --token s6h2dw.haizxq03751eiv12 --discovery-token-ca-cert-hash sha256:ae7aedb206ddb8bc08867b568443c831672b0b1c00e54b608e51270a160e2904`

添加成功：
```
This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.
```

最后在master机器上执行`kubectl get nodes`：

```sh
root@ubuntu1:~# kubectl get nodes
NAME      STATUS   ROLES    AGE   VERSION
ubuntu1   Ready    master   35m   v1.15.1
ubuntu2   Ready    <none>   81s   v1.15.1
```
node节点添加成功。

## 可能会遇到的问题

[ERROR NumCPU]: the number of available CPUs 1 is less than the required 2

至少需要2个核心，虚拟机可以指定核心数

