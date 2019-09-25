# ceph

# 前期准备

### 首先配置第一台节点，目前假定名称ceph1

首先修改源，所有的ceph节点都要做，这样才能成功下载到程序。

把下面的源增加到/etc/yum.repos.d 下面，注意下面的源是针对ceph luminous版本的。注意增加进去的时候，文件名不要是ceph.repo。

```repo
[Ceph]
name=Ceph packages for $basearch
baseurl=http://mirrors.ustc.edu.cn/ceph/rpm-luminous/el7/$basearch
enabled=1
priority=1
gpgcheck=1
type=rpm-md
gpgkey=http://mirrors.ustc.edu.cn/ceph/keys/release.asc
priority=1

[Ceph-noarch]
name=Ceph noarch packages
baseurl=http://mirrors.ustc.edu.cn/ceph/rpm-luminous/el7/noarch
enabled=1
priority=1
gpgcheck=1
type=rpm-md
gpgkey=http://mirrors.ustc.edu.cn/ceph/keys/release.asc
priority=1

[ceph-source]
name=Ceph source packages
baseurl=http://mirrors.ustc.edu.cn/ceph/rpm-luminous/el7/SRPMS
enabled=1
priority=1
gpgcheck=1
type=rpm-md
gpgkey=http://mirrors.ustc.edu.cn/ceph/keys/release.asc
priority=1

```

将上面的文件内容写入到 /etc/yum.repos.d 目录下，新建一个文件 Ceph.repo

```ddd
cat << EOM > /etc/yum.repos.d/ceph.repo
[ceph-noarch]
name=Ceph noarch packages
baseurl=https://download.ceph.com/rpm-luminous/el7/noarch
enabled=1
gpgcheck=1
type=rpm-md
gpgkey=https://download.ceph.com/keys/release.asc
EOM

```

然后yum update一下

注意防火墙，selinux一定要关

```disable
systemctl stop firewalld
systemctl disable firewalld

关闭selinux
先：setenforce 0
然后修改 /etc/selinux/config 确保重启以后selinux也能保持关闭 

```

安装ceph-deploy
```ss
yum install ceph-deploy
```


解决服务器之间的不用密码互访问题，目前用root做测试，使用下面的命令生成密钥对

```ss
ssh-keygen -t rsa 
```
将生成的公钥id_rsa.pub放到需要免密登陆的目标主机里，写入到authorized_keys文件中，注意authorized_keys的权限应该是g-w，具体操作百度。


准备安装ceph环境，到一个临时目录下，或者建立一个目录叫ceph

环境清理(如果有中间有任何问题，可以用下面的命令清理现场，重新安装)
```ff
ceph-deploy --username root purge ceph1
ceph-deploy --username root purgedata ceph1
ceph-deploy --username root forgetkeys
rm ceph.*
```

# 新节点设置与配置

设置新节点（不管是任何类型的节点，都要这样处理一下）:
```
ceph-deploy --username root new ceph1
```

安装新节点（不管是任何类型的节点，都要这样处理一下）:
```ddd
ceph-deploy --username root install ceph1
```


执行上面命令的过程中可能会遇到错误:

```sdf
Traceback (most recent call last):
  File "/usr/bin/ceph-deploy", line 18, in <module>
    from ceph_deploy.cli import main
  File "/usr/lib/python2.7/site-packages/ceph_deploy/cli.py", line 1, in <module>
    import pkg_resources
ImportError: No module named pkg_resources
```

解决方案:

```ss
wget https://files.pythonhosted.org/packages/5f/ad/1fde06877a8d7d5c9b60eff7de2d452f639916ae1d48f0b8f97bf97e570a/distribute-0.7.3.zip

解压然后：
cd distribute-0.7.3
python setup.py install 

```

设置ceph node

## aaa

Deploy the initial monitor(s) and gather the keys:
创建一些密钥
```ee
ceph-deploy mon create-initial
```

Use ceph-deploy to copy the configuration file and admin key to your admin node and your Ceph Nodes so that you can use the ceph CLI without having to specify the monitor address and ceph.client.admin.keyring each time you execute a command.

把上面生成的各种密钥拷贝到各台机器（管理节点和ceph节点）上去，保证能够管理。

```
ceph-deploy admin node1 node2 node3
```

## 部署管理节点
```ceph
ceph-deploy mgr create node1
```

## 部署ceph节点

注意下面得官方说明， 意思就是你要挂在的卷最好是没用的，没数据的卷。做这一步之前应该完成上面的各种初始化工作，并且key要部署到ceph节点上。

Add three OSDs. For the purposes of these instructions, we assume you have an unused disk in each node called /dev/vdb. Be sure that the device is not currently in use and does not contain any important data.

```osd
ceph-deploy osd create --data /dev/sdb ceph3
```

可以在各台节点上检查健康状态，自己理解下面命令的意义

```
ssh node1 sudo ceph health
```

正常情况下应该返回: HEALTH_OK

执行 ceph -s 可以看到整个集群的状况

```ss
[root@ceph1 temp]# ceph -s
  cluster:
    id:     323e7b25-3c61-4a51-a22c-17ed6b9e6e6c
    health: HEALTH_OK

  services:
    mon: 1 daemons, quorum ceph1
    mgr: ceph1(active)
    osd: 2 osds: 2 up, 2 in

  data:
    pools:   0 pools, 0 pgs
    objects: 0 objects, 0B
    usage:   2.00GiB used, 223GiB / 225GiB avail
    pgs:
```

## 部署RGW节点

```s
ceph-deploy rgw create ceph1
```

部署之后可以看到节点情况发生变化

```d

[root@ceph1 temp]# ceph -s
  cluster:
    id:     323e7b25-3c61-4a51-a22c-17ed6b9e6e6c
    health: HEALTH_WARN
            Degraded data redundancy: 187/561 objects degraded (33.333%), 17 pgs degraded

  services:
    mon: 1 daemons, quorum ceph1
    mgr: ceph1(active)
    osd: 2 osds: 2 up, 2 in
    rgw: 1 daemon active

  data:
    pools:   4 pools, 32 pgs
    objects: 187 objects, 1.09KiB
    usage:   2.01GiB used, 223GiB / 225GiB avail
    pgs:     187/561 objects degraded (33.333%)
             17 active+undersized+degraded
             15 active+undersized

```

创建存储池，不知道下面的命令中最后的8是什么意思，要查查手册？
```create pool
ceph osd pool create mytestpool 8
```

可以用rados相关命令查看存储池中的文件:
```
rados ls --pool=mytestpool
```

创建一个文本文件，比如名叫abc.txt, 写入一些内容，然后写入到存储中。

```write
rados put test-object-1 abc.txt --pool=mytestpool
```

再次执行rados相关命令可以看到已经作为对象存储进池的文件了
```
[root@ceph1 testdir]# rados ls --pool=mytestpool
test-object-1
```

查看相关对象的存储信息：

```dd
[root@ceph1 testdir]# ceph osd map mytestpool test-object-1
osdmap e22 pool 'mytestpool' (5) object 'test-object-1' -> pg 5.74dc35e2 (5.2) -> up ([1,0], p1) acting ([1,0], p1)
```

将文件获取到本地进行测试:

```eee
rados get --pool=mytestpool test-object-1 /root/newdir/new.txt
可以看到获取下来的文件的内容和之前写入的一样
```