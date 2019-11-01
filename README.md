## Prometheus Operator监控Kubernetes集群

**配置：**

| 主机名 |  操作系统版本   |      ip      | docker version | kubelet version | prometheus version |    备注    |
| :----: | :-------------: | :----------: | :------------: | :-------------: | :----------------: | :--------: |
| master | Centos 7.6.1810 | 172.27.9.131 | Docker 18.09.6 |     V1.14.2     |      v2.11.0       | master主机 |
| node01 | Centos 7.6.1810 | 172.27.9.135 | Docker 18.09.6 |     V1.14.2     |      v2.11.0       |  node节点  |
| node02 | Centos 7.6.1810 | 172.27.9.136 | Docker 18.09.6 |     V1.14.2     |      v2.11.0       |  node节点  |

## prometheus简介

`Prometheus`是一个开源系统监控和警报工具包，最初是在soundcloud构建的。自2012年成立以来，许多公司和组织都采用了Prometheus，该项目拥有一个非常活跃的开发人员和用户社区。它现在是一个独立的开源项目，独立于任何公司进行维护，于2016年加入了云原生计算基金会，成为继kubernetes之后的第二个托管项目。



**特点：**

> * 用度量名和键值对识别时间序列数据的多维数据模型
> * 灵活的查询语言
> * 不依赖分布式存储；单服务器节点是自治的
> * 通过http上的pull模型进行时间序列收集
> * 通过中间网关支持推送时间序列
> * 通过服务发现或静态配置发现目标
> * 多种图形和仪表板支持模式



**在微服务架构里，其对多维数据收集和查询有很好的的支持。**

## prometheus架构

![图片.png](https://ask.qcloudimg.com/draft/6211241/69sghwyrmg.png)

Prometheus从jobs获取度量数据，也直接或通过推送网关获取临时jobs的度量数据。它在本地存储所有被获取的样本，并在这些数据运行规则，对现有数据进行聚合和记录新的时间序列，或生成警报。通过Grafana或其他API消费者，可以可视化的查看收集到的数据。

## Prometheus Operator介绍

 Prometheus Operator是CoreOS开发的基于Prometheus的Kubernetes监控方案 

![图片.png](https://ask.qcloudimg.com/draft/6211241/pw0avnhs76.png)



**Prometheus Operator：**`整合Kubernetes和Prometheus的最佳方法`。



 **Prometheus Operator 功能更特点：**

- **创建/销毁:** 在Kubernetes namespace中更容易启动一个prometheus实例，一个特定的应用程序或团队更容易使用Operator。
- **简单配置:** 配置Prometheus的基础，比如versions, persistence, retention policies和来自本机kubernetes资源的副本。
- **通过标签的目标服务:** 基于常见的Kubernetes label查询，自动生成监控target 配置；无需学习prometheus特定的配置语言。



**工作流程：**

![图片.png](https://ask.qcloudimg.com/draft/6211241/t6h3o2ca1g.png)



## Prometheus Operator部署

### 安装文件下载

```bash
[root@master ~]# git clone https://github.com/coreos/kube-prometheus.git
```

![图片.png](https://ask.qcloudimg.com/draft/6211241/euexo8344s.png)

### 镜像下载

**下载镜像：**

```bash
docker pull registry.cn-hangzhou.aliyuncs.com/loong576/configmap-reload:v0.0.1

docker pull registry.cn-hangzhou.aliyuncs.com/loong576/alertmanager:v0.18.0

docker pull registry.cn-hangzhou.aliyuncs.com/loong576/kube-state-metrics:v1.8.0

docker pull registry.cn-hangzhou.aliyuncs.com/loong576/kube-rbac-proxy:v0.4.1

docker pull registry.cn-hangzhou.aliyuncs.com/loong576/node-exporter:v0.18.1

docker pull registry.cn-hangzhou.aliyuncs.com/loong576/k8s-prometheus-adapter-amd64:v0.5.0

docker pull registry.cn-hangzhou.aliyuncs.com/loong576/prometheus-config-reloader:v0.33.0

docker pull registry.cn-hangzhou.aliyuncs.com/loong576/prometheus:v2.11.0

docker pull registry.cn-hangzhou.aliyuncs.com/loong576/prometheus-operator:v0.33.0

docker pull registry.cn-hangzhou.aliyuncs.com/loong576/grafana:6.4.3

```

**打tag：**

```bash
docker tag  registry.cn-hangzhou.aliyuncs.com/loong576/configmap-reload:v0.0.1 quay.io/coreos/configmap-reload:v0.0.1

docker tag registry.cn-hangzhou.aliyuncs.com/loong576/alertmanager:v0.18.0 quay.io/prometheus/alertmanager:v0.18.0

docker tag registry.cn-hangzhou.aliyuncs.com/loong576/kube-state-metrics:v1.8.0 quay.io/coreos/kube-state-metrics:v1.8.0

docker tag registry.cn-hangzhou.aliyuncs.com/loong576/kube-rbac-proxy:v0.4.1 quay.io/coreos/kube-rbac-proxy:v0.4.1

docker tag registry.cn-hangzhou.aliyuncs.com/loong576/node-exporter:v0.18.1  quay.io/prometheus/node-exporter:v0.18.1

docker tag  registry.cn-hangzhou.aliyuncs.com/loong576/k8s-prometheus-adapter-amd64:v0.5.0 quay.io/coreos/k8s-prometheus-adapter-amd64:v0.5.0

docker tag  registry.cn-hangzhou.aliyuncs.com/loong576/prometheus-config-reloader:v0.33.0 quay.io/coreos/prometheus-config-reloader:v0.33.0

docker tag registry.cn-hangzhou.aliyuncs.com/loong576/prometheus:v2.11.0 quay.io/prometheus/prometheus:v2.11.0

docker tag registry.cn-hangzhou.aliyuncs.com/loong576/prometheus-operator:v0.33.0  quay.io/coreos/prometheus-operator:v0.33.0

docker tag registry.cn-hangzhou.aliyuncs.com/loong576/grafana:6.4.3 grafana/grafana:6.4.3
```

**删除镜像：**

```bash
docker rmi -f  registry.cn-hangzhou.aliyuncs.com/loong576/configmap-reload:v0.0.1

docker rmi -f registry.cn-hangzhou.aliyuncs.com/loong576/alertmanager:v0.18.0

docker  rmi -f  registry.cn-hangzhou.aliyuncs.com/loong576/kube-state-metrics:v1.8.0

docker  rmi -f  registry.cn-hangzhou.aliyuncs.com/loong576/kube-rbac-proxy:v0.4.1

docker  rmi -f  registry.cn-hangzhou.aliyuncs.com/loong576/node-exporter:v0.18.1

docker  rmi -f  registry.cn-hangzhou.aliyuncs.com/loong576/k8s-prometheus-adapter-amd64:v0.5.0

docker  rmi -f  registry.cn-hangzhou.aliyuncs.com/loong576/prometheus-config-reloader:v0.33.0

docker  rmi -f  registry.cn-hangzhou.aliyuncs.com/loong576/prometheus:v2.11.0

docker  rmi -f  registry.cn-hangzhou.aliyuncs.com/loong576/prometheus-operator:v0.33.0

docker  rmi -f  registry.cn-hangzhou.aliyuncs.com/loong576/grafana:6.4.3
```

**以上三个步骤所有node节点都执行**

### 执行安装

```bash
[root@master kube-prometheus]# kubectl create -f manifests/setup
[root@master kube-prometheus]# kubectl create -f manifests/
```

### 资源查看

```bash
[root@master kube-prometheus]# kubectl get all -n monitoring 
[root@master kube-prometheus]# kubectl get prometheus --all-namespaces -o wide
```

![图片.png](https://ask.qcloudimg.com/draft/6211241/x9vmpeawpx.png)

所有资源都在monitoring命名空间中。

### 更改访问方式

```bash
[root@master kube-prometheus]# kubectl edit -n monitoring service prometheus-k8s
service/prometheus-k8s edited
[root@master kube-prometheus]# kubectl edit -n monitoring service grafana
service/grafana edited
[root@master kube-prometheus]# kubectl edit -n monitoring service alertmanager-main
service/alertmanager-main edited
```

![图片.png](https://ask.qcloudimg.com/draft/6211241/kayp677trz.png)

分别修改service prometheus-k8s、grafana和alertmanager-main，service类型为NodePort，端口分别为30021、30022、30023

## 页面展示

### 访问 Prometheus 

 [http://172.27.9.131:30021](http://172.27.9.131:30021/) 

![图片.png](https://ask.qcloudimg.com/draft/6211241/u7fm9nefdu.png)

![图片.png](https://ask.qcloudimg.com/draft/6211241/yruirh2lb2.png)

### 访问 Alert Manager 

 [http://172.27.9.131:30023](http://172.27.9.131:30023/) 

![图片.png](https://ask.qcloudimg.com/draft/6211241/tx7hmyh74m.png)
![图片.png](https://ask.qcloudimg.com/draft/6211241/obdkfp2zq0.png)

### 访问 Grafana 

 [http://172.27.9.131:30022](http://172.27.9.131:30022/)  用户名密码都为admin

![图片.png](https://ask.qcloudimg.com/draft/6211241/s9oh33ey1o.png)

**内置模板查看：**

![图片.png](https://ask.qcloudimg.com/draft/6211241/uj7jq204bq.png)

**集群资源查看：**

![图片.png](https://ask.qcloudimg.com/draft/6211241/y1db7586ra.png)

**kubelet查看：**

![图片.png](https://ask.qcloudimg.com/draft/6211241/318km0vsxy.png)

**StatefulSets查看：**

![image-20191101163722782](E:\jianguo\typora-pic\image-20191101163722782.png)

**Pod查看：**

![图片.png](https://ask.qcloudimg.com/draft/6211241/4msdjns10t.png)

**API查看：**

![图片.png](https://ask.qcloudimg.com/draft/6211241/rb5zq0s8fm.png)

**namespace查看：**

![图片.png](https://ask.qcloudimg.com/draft/6211241/eag0mdxjjd.png)



**查看各节点资源使用：**

![图片.png](https://ask.qcloudimg.com/draft/6211241/ap51pbnc59.png)

**Prometheus查看：**

![图片.png](https://ask.qcloudimg.com/draft/6211241/tytehb0is1.png)

**其他模板：**

自带的模板很丰富，不过也可以下载其他模板，比如 ‘1 Node Exporter for Prometheus 监控展示看板 update！’ ：https://grafana.com/api/dashboards/8919/revisions/10/download

![图片.png](https://ask.qcloudimg.com/draft/6211241/l3k6dcv35a.png)

## Prometheus Operator卸载

```bash
[root@master kube-prometheus]# kubectl delete --ignore-not-found=true -f manifests/ -f manifests/setup
```

