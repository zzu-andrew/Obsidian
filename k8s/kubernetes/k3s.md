## 4.1 Multipass介绍

[Multipass](https://multipass.run/)是一个轻量级的虚拟机管理工具， 可以用来在本地快速创建和管理虚拟机， 相比于VirtualBox或者VMware这样的虚拟机管理工具， [Multipass](https://multipass.run/)更加轻量快速， 而且它还提供了一些命令行工具来方便我们管理虚拟机。 官方网址: [https://Multipass.run/](https://multipass.run/)

### 安装Multipass

###  Linux  
```bash
sudo snap install multipass
```
### 4.1.2 Multipass常用命令

关于Multipass的一些常用命令我们可以通过`multipass help`来查看， 这里大家只需要记住几个常用的命令就可以了，
```bash
# 查看帮助  
multipass help  
multipass help <command>  
​  
# 创建一个名字叫做k3s的虚拟机  
multipass launch --name k3s  
​  
# 在虚拟机中执行命令  
multipass exec k3s -- ls -l  
​  
# 进入虚拟机并执行shell  
multipass shell k3s  
​  
# 查看虚拟机的信息  
multipass info k3s  
​  
# 停止虚拟机  
multipass stop k3s 
​  
# 启动虚拟机  
multipass start k3s  
​  
# 删除虚拟机  
multipass delete k3s  
​  
# 清理虚拟机  
multipass purge  
​  
# 查看虚拟机列表  
multipass list  

# 挂载目录（将本地的~/kubernetes/master目录挂载到虚拟机中的~/master目录）  
multipass mount ~/kubernetes/master master:~/master  
```

## [k3s](https://k3s.io/)介绍

[k3s](https://k3s.io/) 是一个轻量级的[Kubernetes](https://kubernetes.io/)发行版，它是 [Rancher Labs](https://www.rancher.com/) 推出的一个开源项目， 旨在简化[Kubernetes](https://kubernetes.io/)的安装和维护，同时它还是CNCF认证的[Kubernetes](https://kubernetes.io/)发行版。

### 创建和配置master节点

首先我们需要使用multipass创建一个名字叫做k3s的虚拟机，

```bash
multipass launch --name k3s --cpus 2 --memory 8G --disk 20G
```

虚拟机创建完成之后， 可以配置SSH密钥登录， 不过这一步并不是必须的， 即使不配置也可以通过`multipass exec`或者`multipass shell`命令来进入虚拟机， 然后我们需要在master节点上安装[k3s](https://k3s.io/)，

使用[k3s](https://k3s.io/)搭建kubernetes集群非常简单， 只需要执行一条命令就可以在当前节点上安装[k3s](https://k3s.io/)， 打开刚刚创建的k3s虚拟机， 执行下面的命令就可以安装一个[k3s](https://k3s.io/)的master节点，

```bash
# 安装k3s的master节点 
curl -sfL https://get.k3s.io | sh -
# 国内用户可以换成下面的命令，使用ranher的镜像源来安装：
curl -sfL https://rancher-mirror.rancher.cn/k3s/k3s-install.sh | INSTALL_K3S_MIRROR=cn sh -
```

安装完成之后，可以通过`kubectl`命令来查看集群的状态，

```bash
sudo kubectl get nodes
```
### 创建和配置worker节点

接下来需要在这个master节点上获取一个token， 用来作为创建worker节点时的一个认证凭证， 它保存在`/var/lib/rancher/k3s/server/node-token`这个文件里面， 我们可以使用`sudo cat`命令来查看一下这个文件中的内容，

```bash
sudo cat /var/lib/rancher/k3s/server/node-token
```

将TOKEN保存到一个环境变量中

```bash
TOKEN=$(multipass exec k3s sudo cat /var/lib/rancher/k3s/server/node-token)
```

保存master节点的IP地址

```bash
MASTER_IP=$(multipass info k3s | grep IPv4 | awk '{print $2}')
```
确认：

```bash
echo $MASTER_IP
```
使用刚刚的`TOKEN`和`MASTER_IP`来创建两个worker节点 并把它们加入到集群中

```bash
# 创建两个worker节点的虚拟机  
multipass launch --name worker1 --cpus 2 --memory 8G --disk 10G  
multipass launch --name worker2 --cpus 2 --memory 8G --disk 10G  
  
# 在worker节点虚拟机上安装k3s  
 for f in 1 2; do  
     multipass exec worker$f -- bash -c "curl -sfL https://rancher-mirror.rancher.cn/k3s/k3s-install.sh | INSTALL_K3S_MIRROR=cn K3S_URL=\"https://$MASTER_IP:6443\" K3S_TOKEN=\"$TOKEN\" sh -"  
 done
```

或者手动单行执行安装

```bash
multipass exec worker2 -- bash -c "curl -sfL https://rancher-mirror.rancher.cn/k3s/k3s-install.sh | INSTALL_K3S_MIRROR=cn K3S_URL=\"https://$MASTER_IP:6443\" K3S_TOKEN=\"$TOKEN\" sh -"
```


这样就完成了一个多节点的kubernetes集群的搭建。

## Portainer的安装和使用

[Portainer](https://www.portainer.io/) 是一个轻量级的容器管理工具， 可以用来管理Docker和Kubernetes， 它提供了一个Web界面来方便我们管理容器， 官方网址: [https://www.portainer.io/](https://www.portainer.io/)

### 7.1 安装Portainer

```bash
# 创建一个名字叫做portainer的虚拟机  
multipass launch --name portainer --cpus 2 --memory 8G --disk 10G
```

当然也可以直接安装在我们刚刚创建的master节点上，
```bash
# 在master节点上安装portainer，并将其暴露在NodePort 30777上  
kubectl apply -n portainer -f https://downloads.portainer.io/ce2-19/portainer.yaml
```

或者使用Helm安装

```bash
# 使用Helm安装Portainer  
helm upgrade --install --create-namespace -n portainer portainer portainer/portainer --set tls.force=true
```

然后直接访问 `https://localhost:30779/` 或者 `http://localhost:30777/` 就可以了，

## 8. Helm的安装和使用

[Helm](https://helm.sh/) 是一个Kubernetes的包管理工具， 可以用来管理Kubernetes的应用， 它提供了一个命令行工具来方便我们管理Kubernetes的应用， 官方网址: [https://helm.sh/](https://helm.sh/)

### 8.1 安装Helm

使用包管理器安装：

# macOS  
brew install helm  
  
# Windows  
choco install kubernetes-helm  
# 或者  
scoop install helm  
  
# Linux（Debian/Ubuntu）  
curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null  
sudo apt-get install apt-transport-https --yes  
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list  
sudo apt-get update  
sudo apt-get install helm  
  
# Linux（CentOS/Fedora）  
sudo dnf install helm  
  
# Linux（Snap）  
sudo snap install helm --classic  
  
# Linux（FreeBSD）  
pkg install helm  
  


使用脚本安装

$ curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3  
$ chmod 700 get_helm.sh  
$ ./get_helm.sh

或者

curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash



