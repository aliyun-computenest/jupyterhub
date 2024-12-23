# Jupyterhub服务实例部署文档

## 概述

支持多用户的 Jupyter Notebook 服务器，用于创建、管理、代理多个 Jupyter Notebook 实例。具有扩展性和可定制性，服务本身免费，只用为云服务资源付费，欢迎大家使用。
## 计费说明
Jupyterhub部署的为社区开源版本，源码参考[Github Repo](https://github.com/jupyterhub/zero-to-jupyterhub-k8s)
- 已有阿里云ack集群，这种情况下可以直接将服务部署到该集群中，用户不需付费。
- 新建阿里云ack集群，然后部署服务，这种情况下只用支付ack资源本身的费用。

Jupyterhub在计算巢上的费用主要涉及：

- 所选vCPU与内存规格
- 磁盘容量
- 公网带宽
- ack集群费用

计费方式包括：

- 按量付费（小时）
- 包年包月

预估费用在创建实例时可实时看到。

## 部署架构
Jupyterhub服务为容器服务，部署在ack集群上。

## RAM账号所需权限
Jupyterhub服务需要对ECS、VPC、ACK等资源进行访问和创建操作，若您使用RAM用户创建服务实例，需要在创建服务实例前，对使用的RAM用户的账号添加相应资源的权限。添加RAM权限的详细操作，请参见[为RAM用户授权](https://help.aliyun.com/document_detail/121945.html)。所需权限如下表所示。

| 权限策略名称                          | 备注                         |
|---------------------------------|----------------------------|
| AliyunECSFullAccess             | 管理云服务器服务（ECS）的权限           |
| AliyunVPCFullAccess             | 管理专有网络（VPC）的权限             |
| AliyunROSFullAccess             | 管理资源编排服务（ROS）的权限           |
| AliyunComputeNestUserFullAccess | 管理计算巢服务（ComputeNest）的用户侧权限 |
| AliyunCloudMonitorFullAccess    | 管理云监控（CloudMonitor）的权限     |
| AliyunCSFullAccess              | 管理容器服务(CS)的权限              |

## 部署流程
### 部署步骤
您可以在阿里云计算巢自行搜索，也可以通过下述部署链接快速到达。

[部署链接](https://computenest.console.aliyun.com/service/instance/create/cn-hangzhou?type=user&ServiceName=Jupyterhub%E7%A4%BE%E5%8C%BA%E7%89%88)

### 部署参数说明
您在创建服务实例的过程中，需要配置服务实例信息。下文介绍Jupyterhub服务实例输入参数的详细信息,分为已有ack集群和新建ack集群两种。
#### 已有ack集群
![create-1](create-1.png)
![create-1](create-2.png)
![create-1](create-3.png)

是否新建ack集群参数选择否时，代表现在已有ack集群，此时需要填写以下参数。

| 参数组       | 参数项       | 示例                              | 说明                           |
|-----------|-----------|---------------------------------|------------------------------|
| 服务实例名称    |           | test                            | 实例的名称                        |
| 地域        |           | 华东1（杭州）                         | 选中服务实例的地域，建议就近选中，以获取更好的网络延时。 |
| 是否新建ack集群 | 是否新建ack集群 | 否                               | 选择否代表已有ack集群，不用新建            |
| 是否新建ack集群 | K8s集群ID   | ccde6deb0f612402786e611a7e1230d | 根据地域选择地域中用户已有的集群id           |
| 应用配置       | 域名        | jupyter.mycompany.com        | 访问的域名 |
| 应用配置       | Helm 配置    | {}                          | helm的配置详细参考https://github.com/jupyterhub/zero-to-jupyterhub-k8s)｜

#### 新建ack集群

| 参数组          | 参数项               | 示例            | 说明                                                                                                                                    |
|--------------|-------------------|---------------|---------------------------------------------------------------------------------------------------------------------------------------|
| 服务实例名称       |                   | test          | 实例的名称                                                                                                                                 |
| 地域           |                   | 华东1（杭州）       | 选中服务实例的地域，建议就近选中，以获取更好的网络延时。                                                                                                          |
| 是否新建ack集群    |                   | 是             | 选择是代表新建ack集群                                                                                                                          |
| 付费类型配置       | 付费类型              | 按量付费 或 包年包月   |                                                                                                                                       |
| 基础配置         | 可用区               | 可用区I          | 地域下的不同可用区域                                                                                                                            |
| 基础配置         | 专有网络VPC实例ID       | vpc-xxx       | 选择地域下可用的vpc,不存在可以新建                                                                                                                   |
| 基础配置         | 交换机实例ID           | vsw-xxx       | 选择vpc下的vsw，这个vsw筛选会受上面不同可用区域影响，不存在可以新建                                                                                                |
| 基础配置         | 实例密码              | ********      | 设置实例密码。长度8~30个字符，必须包含三项（大写字母、小写字母、数字、()~!@#$%^&*-+={}[]:;'<>,.?/ 中的特殊符号）                                                              |
| Kubernetes配置 | Worker节点规格        | 	ecs.g6.large | 选择对应cpu核数和内存大小的ecs实例，用作k8s节点                                                                                                          |
| Kubernetes配置 | Worker 系统盘磁盘类型    | ESSD云盘        | 选择k8s集群Worker节点使用的系统盘磁盘类型                                                                                                             |
| Kubernetes配置 | Worker节点系统盘大小(GB) | 120           | 设置Worker节点系统盘大小，单位为GB                                                                                                                 |
| Kubernetes配置 | ack网络插件           | Flannel       | ack集群对应的网络插件，可以选择Flannel或者Terway，网络插件不同，下面设置的pod网络参数不同                                                                                |
| Kubernetes配置 | Pod 网络 CIDR       | 10.0.0.0/16   | ack Pod网络段，网络插件为Flannel时必填，请填写有效的私有网段，即以下网段及其子网：10.0.0.0/8，172.16-31.0.0/12-16，192.168.0.0/16，不能与 VPC 及 VPC 内已有 Kubernetes 集群使用的网段重复。 |
| Kubernetes配置 | pod交换机实例ID        | vsw-xx        | ack Pod交换机实例id，网络插件为Terway时必填，建议选择网段掩码不大于 19 的虚拟交换机                                                                                   |
| Kubernetes配置 | Service CIDR      | 172.16.0.0/16 | ack Service网络段, 可选范围：10.0.0.0/16-24，172.16-31.0.0/16-24，192.168.0.0/16-24,不能与 VPC 及 VPC 内已有 Kubernetes 集群使用的网段重复。                     |
| 应用配置       | 域名        | jupyter.mycompany.com        | 访问的域名 |
| 应用配置       | Helm 配置    | {}                          | helm的配置详细参考https://github.com/jupyterhub/zero-to-jupyterhub-k8s)｜


### 验证结果
1.查看服务实例，服务实例创建成功后，部署时间大约需要10分钟。部署完成后，页面上可以看到对应的服务实例。

![create-4.png](create-4.png)

2.点击详情，可以查看实例详情，具体页面如下：

![create-5.png](create-5.png)

3.部署成功的服务实例详情如上图所示，需要配置host内容到域名A记录或者配置本机的host配置，最后访问域名即可链接服务
![success-0.png](success-0.png)

## 高级选项
### ACK场景下使用ECI+Spot弹性运行Worker节省成本

选择对应的ack集群，运维管理->组件管理->安装ACK Virtual Node 开启ECI弹性能力

进入服务实例->修改配置
![update-1.png](update-1.png)
![update-2.png](update-2.png)


```yaml
singleuser:
  extraLabels:
    alibabacloud.com/eci: 'true'
  extraAnnotations:
    k8s.aliyun.com/eci-spot-strategy: SpotAsPriceGo

```
![success-1.png](success-1.png)
![success-2.png](success-2.png)

### helm包路径
```
https://github.com/aliyun-computenest/jupyterhub/blob/main/targets/jupyterhub-4.0.0.tgz
```

### 更多配置

请参考地址
https://z2jh.jupyter.org/en/stable/jupyterhub/customizing/user-environment.html
