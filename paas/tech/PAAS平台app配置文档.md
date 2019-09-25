#PaaS平台应用配置

###一.cjfh中日友好

####1.zkserver服务
zkserver:

| 服务名称 | 镜像地址 | 容器端口 | 端口号 |
| ------ | ------ | ------ |------ |
| 自定义 | harbor.cechealth.cn/platform/cjfh-zk:0.2 | 2181 | 自定义 |
####2.mysql服务
mysql:

| 服务名称 | 镜像地址 | 容器端口 | 端口号 |
| ------ | ------ | ------ |------ |
| 自定义 | harbor.cechealth.cn/platform/cjfh-mysql:0.0.1 | 3306 | 自定义 |
####3.backend服务员
cjfh-backend:

| 服务名称 | 镜像地址 | 容器端口 | 端口号 | 服务依赖 | 环境变量 |
| ------- | ------- | ------- | ----- | ------- | --------|
| 自定义 | harbor.cechealth.cn/platform/cjfh-backend:0.0.1-RELEASE | 8888 | 自定义 | zk服务 | env |

env：

| key | value |
| ------- | ------- |
| CJFH_ZKSERVER_HOST | zk服务名 | 
| CJFH_ZKSERVER_NODE |   cjfh |
| CJFH_ZKSERVER_PORT | zk服务端口号 |
####4.frontend服务
cjfh-frontend:

| 服务名称 | 镜像地址 | 容器端口 | 主服务 | 
| ------- | ------- | ------- | ----- | 
| 自定义 | harbor.cechealth.cn/platform/cjfh-frontend:0.0.1 | 80 | 是

###二.KOL
####1.mysql服务
mysql：

| 服务名称 | 镜像地址 | 容器端口 | 端口号 |
| ------ | ------ | ------ |------ |
| 自定义 | harbor.cechealth.cn/platform/kol-mysql-base-5.7:0.0.1-RELEASE | 3306 | 自定义 |

####2.后端服务
backend:

| 服务名称 | 镜像地址 | 容器端口 | 端口号 | 服务依赖 | 环境变量 |
| ------- | ------- | ------- | ------ | ------- | ------- |
| kolbackend | harbor.cechealth.cn/platform/kol-backend-k8s:0.0.9-RELEASE | 80 | 10014 | mysql服务名 | env |

env:

| key | value |
| ------- | ------- |
| MYSQL_SERVICE_HOST | mysql服务名 | 
| MYSQL_SERVICE_PORT | mysql服务端口号 |

####3.前端服务
frontend:

| 服务名称 | 镜像地址 | 容器端口 | 主服务 | 服务依赖 |
| ------- | ------- | ------- | ------ | ------- | 
| 自定义 | harbor.cechealth.cn/platform/kol-frontend:0.0.13-RELEASE | 80 | 是 | 后端服务名 |



