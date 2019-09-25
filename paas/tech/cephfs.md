# cephfs 文件系统
Ceph自诞生之日起，就被定位为一个分布式文件系统。时至今日，虽然Ceph的三大典型应用场景中，RBD和RGW先后乘着云计算的东风后来
居上获得了日益广泛的应用，但是起步最早的CephFS反而一直迟迟未能有所建树。究其原因，一是文件系统采用树状结构管理数据（文件和目录）、
基于查表进行寻址的设计理念，与Ceph采用扁平方式管理数据、基于计算进行寻址的设计理念格格不入；二是支持文件系统必然要求Ceph引入集中的元数据
管理服务器（作为树状结构的统一入口用于寻址），这又与Ceph去中心化、追求近乎无限横向扩展能力的设计思想激烈冲突。

尽管颇具戏剧性，然而一个不可否认的事实是：RBD和RGW的蓬勃发展反过来又促使Ceph在云计算以外的领域也迅速普及并逐渐变得广为人知。众所周知，文件系统
中元数据的组织和索引方式极大程度上决定了文件系统的功能和性能。CephFS基于MDS（MetaData Server）对元数据进行管理。容易理解。Ceph的分布式基因
使得MDS管理元数据的方式注定与传统文件存储不同，而是有自身特点：
+ 采用多实例消除性能瓶颈和提升可靠性。
+ 采用大型日志文件和延迟删除日志机制提升元数据读写性能。
+ 将Inode内嵌至Dentry中来提升文件索引效率。
+ 采用目录分片重新定义命名空间层次结构，并且目录分片可以在MDS实例之间动态迁移，从而实现细粒度的流控和负载均衡机制。
### 为什么用cephfs，不用ceph典型应用场景RBD和RGW

## cephfs文件系统的使用

+ 首先你要搭建一个ceph集群。如何搭建ceph集群请参考[ceph搭建](http://172.16.55.10:10013/installation/ceph.html)
。如果要使用cephfs文件系统，则必须要有管理文件元数据的mds节点。
[ceph集群](/paas/images/ceph-s.png)
+ 在集群上创建文件系统
 ``` 
 [root@ceph-slave1 ~]# ceph fs new cephfs2 cephfs_metadata cephfs_data
 *** DEVELOPER MODE: setting PATH, PYTHONPATH and LD_LIBRARY_PATH ***
 new fs with metadata pool 2 and data pool 1
 ```
 
   fs new 表示需要新建一个新的文件系统
   
   cephfs2 表示新的文件系统的名字叫做ceph2
   
   cephfs_metadata 表示文件系统元数据保存信息的存储pool
   
   cephfs_data 表示文件系统的数据保存信息的存储pool。
   
   如果我再新建一个文件系统cephfs3，则会出现如下提示；
   
   ```
   [root@ceph-slave1 ~]# ceph fs new cephfs3 fsmeta fsdata
   *** DEVELOPER MODE: setting PATH, PYTHONPATH and LD_LIBRARY_PATH ***
   Error EINVAL: A filesystem already exists, use `ceph fs rm` if you wish to delete it
   ```
   这就表明ceph集群上只能创建一个文件系统
   
+ 客户端上的挂载

说明：网络系统文件想要使用必须要在本地进行mount操作，mount操作后，即可像本地目录一样操作。ceph的rbd块设备在使用时提供了两种使用方式 librbd和kernel rbd，同样的cephfs也提供了两种使用方式，一种是基于用户空间的文件
系统
   
   
 