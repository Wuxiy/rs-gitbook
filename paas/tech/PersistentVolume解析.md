##Kubernetes中的Persistent Volume解析
###1.概念对比

| PersistentVolume | PersistentVolumeClaim |
| ---------------- | -----------------|
| 是由管理员设置的存储 | 是用户存储的请求,消耗pvc资源 |
注：pvc是对pv使用的声明，PVC 和 PV 的设计，其实跟“面向对象”的思想完全一致。PVC可以理解为持久化存储的的“接口”，它提供了对某种持久化存储的描述，但不提供具体实现；而这个持久化存储的实现部分则是由PV负责完成。

###2.pv和pvc的生命周期
pv和pvc遵循这样的生命周期：

```
Provisioning ----> Binding ----> Using ----> Releasing ----> Recycling
```
PV在生命周期中，可能处于以下四个阶段之一

- Availablie：可用状态，还未与某个PVC绑定
- Bound：已与某个PVC绑定
- Released：绑定的PVC已经删除，资源已释放，但没有被集群回收
- Failed：自动资源回收失败

下面将用kol-mysql在kubernetes环境中的PersistentVolume和PersistentVolumeClaim做为例子进行名词解释

kolmysql PV 使用yaml描述如下：

```
[root@master ~]# kubectl get pv kolmysql-73 -o yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  annotations:
    pv.kubernetes.io/bound-by-controller: "yes"
  creationTimestamp: 2018-12-27T09:37:14Z
  labels:
    name: kolmysql-73
  name: kolmysql-73
  resourceVersion: "5280271"
  selfLink: /api/v1/persistentvolumeskolmysql-73
  uid: fe57afa3-09ba-11e9-b6b4-fa163ee2ae51
spec:
  accessModes:
  - ReadWriteMany
  capacity:
    storage: 1Gi
  cephfs:
    monitors:
    - 192.168.16.50:6789
    - 192.168.16.51:6789
    - 192.168.16.53:6789
    path: /data/kolmysql/73
    secretRef:
      name: ceph-secret-client-cephfs
    user: admin
  claimRef:
    apiVersion: v1
    kind: PersistentVolumeClaim
    name: kolmysql-pvc
    namespace: "73"
    resourceVersion: "5280269"
    uid: fe5aeba7-09ba-11e9-b6b4-fa163ee2ae51
  persistentVolumeReclaimPolicy: Retain
status:
  phase: Bound
```
kolmysql PVC使用yaml描述如下：

```
[root@master ~]# kubectl get pvc -n 73 kolmysql-pvc -o yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  annotations:
    pv.kubernetes.io/bind-completed: "yes"
    pv.kubernetes.io/bound-by-controller: "yes"
  creationTimestamp: 2018-12-27T09:37:14Z
  name: kolmysql-pvc
  namespace: "73"
  resourceVersion: "5280273"
  selfLink: /api/v1/namespaces/73/persistentvolumeclaims/kolmysql-pvc
  uid: fe5aeba7-09ba-11e9-b6b4-fa163ee2ae51
spec:
  accessModes:
  - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
  selector:
    matchLabels:
      name: kolmysql-73
  storageClassName: ""
  volumeName: kolmysql-73
status:
  accessModes:
  - ReadWriteMany
  capacity:
    storage: 1Gi
  phase: Bound
```
####2.1Provisioning(配置)：
有两种方式来配置 PV：`静态`或`动态`。

`静态`:集群管理员创建一些 PV。它们带有可供群集用户使用的实际存储的细节。它们存在于 Kubernetes API 中，可用于消费。

`动态`:当管理员创建的静态 PV 都不匹配用户的 PersistentVolumeClaim 时，集群可能会尝试动态地为 PVC 创建卷。此配置基于 StorageClasses.PVC 必须请求存储类，并且管理员必须创建并配置该类才能进行动态创建。声明该类为 "" 可以有效地禁用其动态配置。

目前在我们的系统中使用的是静态创建PV，上面的kolmysql-pvc PVC对象的yaml描述中*storageClassName*的值为空""也是为了避免其动态创建PV。目前我们使用这种策略的原因是k8s官方不支持对cephfs持久化卷的动态创建，后期迭代中可以使用cephfs-provisioner第三方插件完成对动态创建PV的支持。

在kolmysql pv的yaml文件中*cephfs*字段里配置了很多ceph相关配置项

```
cephfs:
    monitors:
    - 192.168.16.50:6789
    - 192.168.16.51:6789
    - 192.168.16.53:6789
    path: /data/kolmysql/73
    secretRef:
      name: ceph-secret-client-cephfs
    user: admin
```
* monitors：ceph集群
* path：cephfs文件系统中数据存储路径，/data/kolmysql(隔离不同应用)/73(隔离不同用户)
* secretRef：操作ceph的安全凭证，在代码中的处理是创建deployment的时在对应用户的namespace下创建ceph凭证，赋予改租户使用cephfs的权限

```
[root@master ~]# kubectl get secret -n 74 ceph-secret-client-cephfs -o yaml
apiVersion: v1
data:
  key: QVFDLy9CVmNkZk1NQmhBQTdtMHJFTmYwQmd1ejI5RUdYYmNFbVE9PQ==
kind: Secret
metadata:
  creationTimestamp: 2018-12-27T06:46:17Z
  name: ceph-secret-client-cephfs
  namespace: "74"
  resourceVersion: "5261737"
  selfLink: /api/v1/namespaces/74/secrets/ceph-secret-client-cephfs
  uid: 1c7db387-09a3-11e9-b6b4-fa163ee2ae51
type: kubernetes.io/rbd
```
* user: 操作ceph的用户

####2.2Binding(绑定)：
在用户定义好PVC后，系统将根据PVC对存储资源的请求 (存储空间和访问模式)在已存在的PV中选择一个满足PVC要求的PV，一旦找到，就将该PV与用户定义的PVC进行绑定，然后用户的应用就可以使用这个PVC了。如果系统中没有满足PVC要求的PV，PVC则会无限期处于Pending状态，直到等到系统管理员创建了一个符合要求的PV。PV一旦绑定在某个PVC上，就被这个PVC独占，不能再与其他PVC进行绑定了。在这种情况下，当PVC申请的存储空间比PV的少时，整个PV的空间都能够为PVC所用，可能会造成资源的浪费。如果资源供应使用的是动态模式，则系统在PVC找到合适的StorageClass后，将会自动创建PV并完成PVC的绑定。

kolmysql-pvc中的*selector*字段：

```
selector:
    matchLabels:
      name: kolmysql-73
```
匹配label为*name: kolmysql-73*的pv，PVC通过selector绑定PV，建立绑定关系后PV也会显示该引用关系

```
claimRef:
    apiVersion: v1
    kind: PersistentVolumeClaim
    name: kolmysql-pvc
    namespace: "73"
    resourceVersion: "5280269"
    uid: fe5aeba7-09ba-11e9-b6b4-fa163ee2ae51
```
PV的*claimRef*字段显示在命名空间为73的kolmysql-pvc引用了该PV。PV和PVC是OneToOne的关系，PVC绑定PV后，VP就会排斥其他的PVC对其绑定。
####2.3Using(使用)：
Pod 使用volume的定义，将PVC挂载到容器内的某个路径进行使用。volume的类型为persistentVoulumeClaim，在容器应用挂载了一个PVC后，就能被持续独占使用。不过，多个Pod可以挂载同一个PVC，应用程序需要考虑多个实例共同访问一块存储空间的问题。

PV和PVC的*accessModes*字段指定了资源的访问方式，受限于存储底层的支持(cephfs支持这三种)，访问方式包括：

| accessModes | mode |
| ---- | ---- |
| ReadWriteOnce | 被单个节点mount为读写rw模式 |
| ReadOnlyMany | 被多个节点mount为只读ro模式 |
| ReadWriteMany | 被多个节点mount为读写rw模式 |
必须要注意的一个关键点就是：即使一个卷支持多种访问模式但是在同一时间只能使用其中一种。

注：节点指Kubernetes集群中的node
####2.4Releasing(释放)
当用户完成对卷的使用时，就可以利用API删除PVC对象了，而且他还可以重新申请。删除PVC后，对应的卷被视为“被释放”，但是这时还不能给其他的PVC使用。之前的PVC数据还保存在卷中，要根据策略来进行后续处理。

####2.5Recycling(回收)

当前的回收策略包括：

- Retain（保留）——手动回收
- Recycle（回收）——基本擦除（rm -rf /thevolume/*）
- Delete（删除）——关联的存储资产（例如 AWS EBS、GCE PD、Azure Disk 和 OpenStack Cinder 卷）将被删除

目前系统中都是默认Retain回收策略，在测试Delete回收策略时目前系统并不支持。
