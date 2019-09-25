# ceph

## ceph luminous

### ceph-mgr

+ ceph luminous版本中新增加了一个组件：Ceph Manager Daemon，简称ceph-mgr。该组件的主要作用是分担和扩展monitor的部分功能，
减轻monitor的负担，让更好地管理ceph存储系统。
+ ceph-mgr是一个新的后台进程，这是任何Ceph部署的必须部分。虽然当ceph-mgr停止时，IO可以继续，但是度量不不会刷新，并且某些与度量相关的请求（例如，ceph df）可能会被阻止。我们建议多部署ceph-mgr的几个实例来实现可靠性。
+ ceph-mgr 守护进程daemon包括基于REST的API管理。注：API仍然是实验性质的，目前有一些限制，但未来会成为API管理的基础。
+ ceph-mgr 还包括一个**Prometheus**插件
+ ceph-mgr 现在有一个**Zabbix**插件。使用zabbix_sender,它可以将集群故障事件发送到Zabbix Server主机。这样可以方便地监视Ceph集群的状态，并在发生故障时发送通知。

#### dashboard的配置

##### 开启监控模块

在`/etc/ceph/ceph.conf`中添加

```
[mgr]
mgr modules = dashboard
```
设置dashboard的ip和端口

```
ceph config-key put mgr/dashboard/server_addr 192.168.16.50
ceph config-key put mgr/dashboard/server_port 7000

```
这个从代码上看应该是可以支持配置文件方式的设置，目前还没看到具体的文档，先按这个设置即可，默认的端口是7000。


重启mgr服务

```
[root@ceph1 ~]# systemctl restart ceph-mgr@ceph1

```

检查端口

```
[root@ceph1 ~]# netstat -tunlp|grep 7000
tcp        0      0 192.168.16.50:7000      0.0.0.0:*               LISTEN      43187/ceph-mgr
```
##### 访问界面
[访问路径](http://192.168.16.50:7000)

![cephfs监控](/paas/images/ceph_dashboard.png)



#### ceph restful api

##### 什么是RESTful插件
Ceph Manager（Ceph-mgr）的RESTful插件提供了一个用于与Red Hat Ceph存储集群交互的API。
可以使用该API来：
1. 显示有关监视器和OSD的信息
2. 创建或编辑池
3. 查看并启动OSD上的预定流程
4. 显示群集，监视器和OSD的配置选项

##### 启用和保护RESTful插件
RESTful插件通过SSL加密连接提供REST API对集群状态的访问。

###### 先决条件

+ 确保至少有一个ceph-mgr守护程序处于活动状态。
+ 如果使用防火墙，请确保`8003`端口在具有活动`ceph-mgr`后台驻留程序的节点上启用该端口。

###### 程序

在具有管理秘钥环的节点上使用以下命令。

###### 1.启用RESTful插件。

```
[root @ ceph1 ~] # ceph mgr module enable restful
```

###### 2.配置SSL证书

+  所有与restful的连接都使用SSL保护，可以使用以下的命令生成自签名证书

```
ceph  restful  create-self-signed-cert

```

+ 请注意，对于自签名证书，大多数客户端都需要一个标志来允许连接和禁止显示警告信息。例如，如果ceph-mgr守护程序位于同一主机上，则：

```
curl -k https://localhost:8003/
```
+ 要正确保护部署，应使用由组织的证书颁发机构签名的证书。例如，可以使用类似于以下命令生成秘钥对：
```
openssl req -new -nodes -x509 \
  -subj "/O=IT/CN=ceph-mgr-restful" \
  -days 3650 -keyout restful.key -out restful.crt -extensions v3_ca
```
+ 本restful.crt应该再由您组织的CA（认证机构）部署。完成后，可以使用以下命令设置：

```
ceph config set mgr/restful/$name/crt -i restful.crt
ceph config set mgr/restful/$name/key -i restful.key
```
+ 其中$name是ceph-mgr实例的名称（通常是主机名）。如果所有管理器实例都要共享相同的证书，可以省略以下$name的部分：
```
[root@admin ~]# ceph config-key set mgr/restful/crt -i restful.crt
[root@admin ~]# ceph config-key set mgr/restful/key -i restful.key
```
###### 3.配置ip和端口

+ 与任何其他RESTful API端点一样，restful绑定到IP和端口。默认情况下，当前活动的ceph-mgr守护程序将绑定到端口8003以及主机上的任何可用IPv4或IPv6地址。由于每个ceph-mgr主机都有自己的restful实例，因此可能还需要单独配置它们。可以通过配置密钥工具更改IP和端口：
```
ceph config set mgr/restful/$name/server_addr $IP
ceph config set mgr/restful/$name/server_port $PORT
```
+ 其中$name是ceph-mgr守护程序的ID（通常是主机名）,这些设置也可以在群集范围内配置，而不是特定于管理器。例如:
```
ceph config set mgr/restful/server_addr $IP
ceph config set mgr/restful/server_port $PORT
```
+ 如果端口没有配置，restful将默认绑定到8003端口。如果没有配置IP地址，restful将绑定到::，这对应于所有可用的IPv4和IPv6地址。
  
###### 4.LOAD BALANCER
  
  请注意，restful只会在当前处于active状态的manager启动。查询Ceph集群状态来查看哪个manager处于active状态（例如，ceph mgr dump）。为了使API可通过一致的URL访问，无论哪个管理器daemon当前处于active状态，您可能需要在前端设置一个负载平衡器，以将流量引导至任何可用的manager endpoint。
+ 创建HTTP用户并生成HTTP基本身份验证的密码。
```
ceph restful create-key <username>
```
+ 替换<username>为用户名。例如，要创建名为的用户admin：
```
[root @ ceph1~] #ceph restful create-key admin
3ce361b7-97fb-4820-8edc-1090841f078e
```
+ 连接到RESTful插件网页。打开Web浏览器并输入以下URL：
```
HTTPS：// <CEPH-MGR>：8003
```
+ 替换为<ceph-mgr>具有活动ceph-mgr守护程序的节点的IP地址或主机名
```
https：//192.168.16.50:8003
```

###### 5.客户端页面需要输入刚配置的用户名和key，在postman中配置认证证书和Host和CRT file以及Key file，CRT file为restful.crt文件，Key file为restful.key文件


###### 192.168.16.61机器上 admin  7dc38e22-baba-437c-b0ee-585eaa2ac7dd