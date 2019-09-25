# Kubernetes13.2版本安装手册
```test
copy from qinhuasong
```
概述：kubernetes13.2集群环境安装（docker版本为18.06），通过kubeadm工具自动安装，可快速搭建集群。本手册以一个master主节点，
两个node集群节点为例进行说明。

##一、K8S集群环境设计
| 主机定位 | 主机IP | 主机名 | 控制节点 |
| ------ | ----- |-----| ------ |
| 192.168.16.67 | k8s-test-master | 集群节点 | 192.168.16.65 |
| k8s-test-node1 | 集群节点 | 192.168.16.66 | k8s-test-node2 |
备注：所需环境为三台机器，一台为主控制节点master，两台为集群节点node，机器操作系统为CentOS7.6, 操作系统安装过程参见《PAAS应用团队虚拟机安装手册》 。
要求每个节点的IP都能互相ping通；
要求每个节点都能上外网（因为需要通过网络下载镜像）
##二、所有节点主机基础配置
###2.1 修改节点机器名
执行如下命令修改节点主机名为所需名字：
```test
[root@test ~] hostnamectl set-hostname [主机名]
```
举例： hostnamectl set-hostname  k8s-test-master

###2.2 修改节点host文件
执行如下命令，在hosts中添加集群节点的ip和主机名：
```test
[root@test ~] vi /etc/hosts
```
添加如下内容：
```text
192.168.16.67 k8s-test-master
192.168.16.65 k8s-test-node1
192.168.16.66 k8s-test-node2
```
###2.3 禁用节点防火墙
执行如下命令，禁用节点上防火墙（本方式在机器重启后会失效，重启后需要重新执行本语句才可关闭）
```text
[root@test ~] systemctl stop firewalld
[root@test ~] systemctl disable firewalld
```
###2.4 关闭节点SELINUX
执行如下命令，关闭节点上SELINUX
```text
[root@test ~] sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
[root@test ~] setenforce 0
```
验证selinux是否关闭
```text
[root@test ~] vi /etc/selinux/config
```
selinux的值为disabled，如下所示
```text
...
SELINUX=disabled
...
```
###2.5  设置iptables处理链
执行如下命令：
```text
[root@test ~] iptables  -F
[root@test ~] iptables -t nat -F
[root@test ~] iptables –I FORWARD –s 0.0.0.0/0 –d 0.0.0.0/0 –j ACCEPT
```
###2.6 设置时钟同步NTP
执行如下命令:
```text
[root@test ~] yum –y install ntp
[root@test ~] ntpdate pool.ntp.org
[root@test ~] systemctl start ntpd
[root@test ~] systemctl enable ntpd
```
###2.7 设置系统内核参数
执行如下命令：
```text
[root@test ~] vi /etc/sysctl.conf
```
设置如下内容：
```text
net.ipv4.ip_forward=1
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-arptables = 1
vm.swappiness=0
```
###2.8  关闭swap交换空间
执行如下命令：
```text
[root@test ~] swapoff -a
```
###2.9  保存内核参数
执行如下命令:
```text
[root@test ~] sysctl -p
```
##三、所有节点YUM源更新
###3.1  更新 docker 的yum源
执行如下命令：
```text
[root@test ~] wget -P /etc/yum.repos.d https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```
###3.2  更新 kubernetes 的yum源
执行如下命令：
```text
[root@test ~] vi /etc/yum.repos.d/kubernetes.repo
```
设置如下内容：
```text
[kubernetes]
name=Kubernetes Repo
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
enabled=1
```
###3.3 更新节点YUM缓存
执行如下命令
```text
[root@test ~] yum clean all
[root@test ~] yum makecache
[root@test ~] yum update
```
备注：yum update 会更新所有，耗时较长
##四、所有节点安装DOCKER
###4.1  安装docker
执行如下命令：
```text
[root@test ~] yum install docker-ce-18.06.1.ce -y
```
备注：因为当前kubernetes最新版本为1.13.2，支持的docker最新版本为18.06； 不支持最新的docker版本18.09，安装18.09可能会存在问题。因此基于1.13.2版本的kubernetes只能安装18.06版本的docker：
###4.2 启动docker并设置开机自启动
执行如下命令
```text
[root@test ~] systemctl start docker
[root@test ~] systemctl enable docker
[root@test ~] systemctl status docker
[root@test ~] docker -v
```
###4.3  设置docker加速器
执行如下命令
```text
[root@test ~] curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s http://f1361db2.m.daocloud.io
[root@test ~] systemctl restart docker
```
##五、所有节点安装kubeadm和kubelet
###5.1  安装 kubelet kubeadm kubectl
执行如下命令
```text
[root@test ~] yum install -y kubelet kubeadm kubectl
```
或者
```text
[root@test ~] yum install kubelet kubeadm kubectl
```
###5.2 设置开机启动kubelet
执行如下命令
```text
[root@test ~] systemctl start kubelet
[root@test ~] systemctl enable kubelet.service
```
###5.3 验证kubelet状态
执行如下命令
```text
[root@test ~] kubectl version
[root@test ~] systemctl status kubelet
[root@test ~] systemctl enable kubelet.service
```
备注：systemctl status kubelet 查看状态时，activating状态和running状态都是正确的，只有failed状态才是kubelet安装失败。针对activating状态，参见文末的异常说明

执行如下命令，查看结果：
```text
[root@test ~] cat /proc/sys/net/bridge/bridge-nf-call-ip6tables
[root@test ~] cat /proc/sys/net/bridge/bridge-nf-call-ip6tables
```
##六、master节点安装镜像
###6.1 master节点查看所需镜像
执行如下命令：
```text
[root@test ~] kubeadm config images list
```
执行结果如下:
```text
k8s.gcr.io/kube-apiserver:v1.13.2
k8s.gcr.io/kube-controller-manager:v1.13.2
k8s.gcr.io/kube-scheduler:v1.13.2
k8s.gcr.io/kube-proxy:v1.13.2
k8s.gcr.io/pause:3.1
k8s.gcr.io/etcd:3.2.24
k8s.gcr.io/coredns:1.2.6
```
###6.2 master节点下载所需的镜像
执行如下命令:
```text
[root@test ~] docker pull mirrorgooglecontainers/kube-apiserver-amd64:v1.13.2
[root@test ~] docker pull mirrorgooglecontainers/kube-controller-manager-amd64:v1.13.2
[root@test ~] docker pull mirrorgooglecontainers/kube-scheduler-amd64:v1.13.2
[root@test ~] docker pull mirrorgooglecontainers/kube-proxy-amd64:v1.13.2
[root@test ~] docker pull mirrorgooglecontainers/pause-amd64:3.1
[root@test ~] docker pull mirrorgooglecontainers/etcd-amd64:3.2.24
[root@test ~] docker pull carlziess/coredns-1.2.6
```
###6.2 master节点给下载的镜像打TAG标签
执行如下命令:
```text
[root@test ~] docker tag mirrorgooglecontainers/kube-apiserver-amd64:v1.13.2 k8s.gcr.io/kube-apiserver:v1.13.2
[root@test ~] docker tag mirrorgooglecontainers/kube-controller-manager-amd64:v1.13.2 k8s.gcr.io/kube-controller-manager:v1.13.2
[root@test ~] docker tag mirrorgooglecontainers/kube-scheduler-amd64:v1.13.2 k8s.gcr.io/kube-scheduler:v1.13.2
[root@test ~] docker tag mirrorgooglecontainers/kube-proxy-amd64:v1.13.2 k8s.gcr.io/kube-proxy:v1.13.2
[root@test ~] docker tag mirrorgooglecontainers/pause-amd64:3.1 k8s.gcr.io/pause:3.1
[root@test ~] docker tag mirrorgooglecontainers/etcd-amd64:3.2.24 k8s.gcr.io/etcd:3.2.24
[root@test ~] docker tag carlziess/coredns-1.2.6 k8s.gcr.io/coredns:1.2.6
```
备注：这里要打标签打成刚才kubeadm config images list执行结果所需的镜像
##七、master节点初始化集群
###7.1 master通过kubeadm初始化集群
执行如下命令:
```text
[root@test ~] kubeadm init --kubernetes-version=v1.13.2 --pod-network-cidr=10.244.0.0/16  --apiserver-advertise-address=192.168.16.67
```
备注：上述 ip 192.168.130 需要和master端ip 相同， --pod-network-cidr 的设置是因为集群网络使用flannel方案，需要设置，值可与本例相同。如果上述kubeadm安装过程中执行失败，可通过如下命令重置：
kubeadm reset,reset后节点需要加入本集群，需要执行一行的命令
```text
kubeadm join 192.168.16.67:6443 --token np23lq.qct0kwkk306egy3x --discovery-token-ca-cert-hash sha256:07294691577f2d6dc7d67a6fc4a0c553d3caea10a014e9c303967db730ae566b
```
###7.2  master设置集群环境参数
执行如下命令:
```text
[root@test ~] mkdir -p $HOME/.kube
[root@test ~] sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
[root@test ~] sudo chown $(id -u):$(id -g) $HOME/.kube/config
[root@test ~] export KUBECONFIG=/etc/kubernetes/admin.conf
```
###7.3 master安装flannel网络
执行如下命令:
```text
[root@test ~] kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/bc79dd1505b0c8681ece4de4c0d86c5cd2643275/Documentation/kube-flannel.yml
```
##八、其他node节点（非master）下载镜像并打TAG标签
###8.1  node节点下载docker镜像
```text
[root@test ~] docker pull mirrorgooglecontainers/kube-proxy-amd64:v1.13.2
[root@test ~] docker pull mirrorgooglecontainers/pause-amd64:3.1
[root@test ~] docker pull carlziess/coredns-1.2.6
```
###8.2 node节点给docker镜像打tag标签
```text
[root@test ~] docker tag mirrorgooglecontainers/kube-proxy-amd64:v1.13.2 k8s.gcr.io/kube-proxy:v1.13.2
[root@test ~] docker tag mirrorgooglecontainers/pause-amd64:3.1 k8s.gcr.io/pause:3.1
[root@test ~] docker tag carlziess/coredns-1.2.6 k8s.gcr.io/coredns:1.2.6
```
###8.3 其他node节点（非master）加入集群
```text
[root@test ~] kubeadm join 192.168.16.67:6443 --token np23lq.qct0kwkk306egy3x --discovery-token-ca-cert-hash sha256:07294691577f2d6dc7d67a6fc4a0c553d3caea10a014e9c303967db730ae566b
```
备注：此命令是之前在master上执行kubeadm init得到的结果，在任一节点上执行，该节点就会加入指定的集群中

###九、校验kubernetes集群搭建完成
在master节点上执行如下命令查看节点信息
```text
kubectl get nodes
```
##十、FAQ
###10.1   kubelet 的异常状态说明
执行systemctl status kubelet 查看kubelet状态，有如下三种
* running
* activating
* failed

其中，failed状态是安装失败，activating状态和running状态是安装成功。针对failed状态，请做如下排查：
* 是否关闭缓存交换：swapoff –a
* 是否关闭防火墙：`systemctl stop firewalld` `systemctl disable firewalld`
* 是否关闭 SELINUX：`sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config` `setenforce 0`

针对activating状态，说明如下：
```text
The kubelet is now restarting every few seconds, as it waits in a crashloop for kubeadm to tell it what to do.
 This crashloop is expected and normal, please proceed with the next step and the kubelet will start running normally.
```
即在安装kubeadm init 之前kubelet会不断重启，直到kubeadm告诉kubelet需要执行的活动为止，因此，对于master节点，在kubeadm init之后，
再检查kubelet状态，就是running状态；对于其他node节点，在加入集群后，再检查kubelet状态就是running状态了。

###10.2   机器重启后，kubelet 的状态异常
重启后，master节点和node节点的交换分区和防火墙可能会重新打开，导致查看kubelet状态总是activating中，导致集群失败。
可在所有节点执行如下命令后再查看集群状态：
```text
swapoff –a
systemctl stop firewalld
```
然后再查看kubelet状态：:
```text
systemctl status kubelet
```
查看集群状态，所有节点状态为ready即可：
```text
kubectl get nodes
```