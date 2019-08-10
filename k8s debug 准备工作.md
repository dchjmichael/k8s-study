# 环境准备

先安装基本的依赖工具，为了方便，以下都是在root身份下进行操作。

我这边的系统是：Ubuntu 64-bit 18.04。

下面有些工具我用了`axel`进行下载，这个工具可以多线程下载，速度很快。

**Docker**

```sh
apt install docker.io
```

**go**

```sh
axel -n 10 https://dl.google.com/go/go1.12.7.linux-amd64.tar.gz
tar -C /usr/local -xzf go1.12.7.linux-amd64.tar.gz
```
安装完后记得设置`GOPATH`，`GOROOT`环境变量。

**etcd**

```sh
axel -n 5 https://github.com/etcd-io/etcd/releases/download/v3.3.13/etcd-v3.3.13-linux-amd64.tar.gz
tar xvfz etcd-v3.3.13-linux-amd64.tar.gz
cd etcd-v3.3.13-linux-amd64
cp etcd etcdctl  /usr/bin/
```

**openssl**
这个我系统自带了，可以用`openssl`命令测试，没有的话apt装下。


**cfssl**

```sh
go get -u github.com/cloudflare/cfssl/cmd/...
```

**delev**
调试工具，后面得靠他debug啦。

```sh
go get -u -v github.com/derekparker/delve/cmd/dlv
```

# 下载源码

```sh
go get -d k8s.io/kubernetes
cd $GOPATH/src/k8s.io/kubernetes
#后面对该版本进行源码分析
git checkout release-1.15
#后面这个参数不能丢
make all GOGCFLAGS="-N -l"
```

>如果make命令找不到，用apt装下。

编译的时候遇到内存耗尽，我这边用的虚拟机，干脆直接分配了8G，然后编译通过。

然后用下面命令启动本地集群。

```sh
#后面参数是大写的“欧”
./hack/local-up-cluster.sh -O
```

# 调试kubectl

由于kubectl是个客户端命令，执行完成后进程就结束了，因此不能用`dlv attach`进行调试，我这里用`dlv debug`进行的本地调试。如果是后台进程，可以用`attach`命令。

```sh
# -- 后面的内容都会作为kubectl的参数
dlv debug cmd/kubectl/kubectl.go -- run my-nginx --image=nginx --replicas=2 --port=80
```

这样就进入了`dlv`的调试环境，这里要介绍下`dlv`的常用操作。


|命令|缩写|说明|
|:---|:----|:----|
|continue|c|继续执行程序，直到遇到下一个断点|
|break|b|设置一个断点，比如`b cmd/kubectl.go:37`，在该文件的37行设置断点|
|next|n|step over到下一行|
|step|s|step in到里面|
|print|p|打印变量的值。我试了下表达式也可以打印|
|list|ls l|打印当前所在位置的源代码|

还有很多其他命令，具体可以看[文档](https://github.com/go-delve/delve/blob/master/Documentation/cli/README.md)


# Enjoy debugging

好了，环境搭建就完成了，享受探索吧！


# 参考文档
[https://github.com/kubernetes/community/blob/master/contributors/devel/running-locally.md](https://github.com/kubernetes/community/blob/master/contributors/devel/running-locally.md)

[https://github.com/go-delve/delve/blob/master/Documentation/cli/README.md](https://github.com/go-delve/delve/blob/master/Documentation/cli/README.md)

[https://jeremy-xu.oschina.io/2018/08/%E6%90%AD%E5%BB%BAk8s%E7%9A%84%E5%BC%80%E5%8F%91%E8%B0%83%E8%AF%95%E7%8E%AF%E5%A2%83/](https://jeremy-xu.oschina.io/2018/08/%E6%90%AD%E5%BB%BAk8s%E7%9A%84%E5%BC%80%E5%8F%91%E8%B0%83%E8%AF%95%E7%8E%AF%E5%A2%83/)

[https://www.centos.bz/2017/08/ubuntu-16-04-k8s-kubernetes-delve-debug/](https://www.centos.bz/2017/08/ubuntu-16-04-k8s-kubernetes-delve-debug/)