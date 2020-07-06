<style>
    .page-inner {
        max-width: 2000px !important;
    }
</style>
# 第三方软件说明
### 大数据应用平台
| 软件名称 | 版本号 | 依赖环境版本 | 来源 |
| ------ | ------ | ------ |------ |
|docker| 18.06.1-ce |||
|ceph|luminous(12.2.10)|||
|kubernetes|v1.13.2|GoVersion:"go1.11.4"，Compiler:"gc"，Platform:"linux/amd64"，etcd：k8s.gcr.io/etcd:3.2.24，flannel：||
|etcd|3.2.24|||
|flannel|0.10.0|||
|mysql|5.7||docker pull harbor.xxxx.cn/confluence/mysql:5.7|
|prometheus|2.4.3||https://github.com/prometheus/prometheus/releases/download/v2.4.3/prometheus-2.4.3.linux-amd64.tar.gz|
|alertmanager|0.15.2||https://github.com/prometheus/alertmanager/releases/download/v0.15.2/alertmanager-0.15.2.linux-amd64.tar.gz|
|grafana|5.2.4||https://s3-us-west-2.amazonaws.com/grafana-releases/release/grafana-5.2.4.linux-amd64.tar.gz|
|node_exporter|0.16.0||https://github.com/prometheus/node_exporter/releases/download/v0.16.0/node_exporter-0.16.0.linux-amd64.tar.gz||