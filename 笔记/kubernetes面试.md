# kubernetes面试

### 什么是 Kubernetes

Kubernetes 是一个全新的基于容器技术的分布式系统支撑平台。是 Google 开源的容器集群管理系统（谷歌内部：Borg）。在 Docker 技术的基础上，为容器化的应用提供部署运行、资源调度、服务发现和动态伸缩等一系列完整功能，提高了大规模容器集群管理的便捷性。并且具有完备的集群管理能力，多层次的安全防护和准入机制、多租户应用支撑能力、透明的服务注册和发现机制、內建智能负载均衡器、强大的故障发现和自我修复能力、服务滚动升级和在线扩容能力、可扩展的资源自动调度机制以及多粒度的资源配额管理能力。

###  Kubernetes 和 Docker 的关系

Docker 提供容器的生命周期管理和 Docker 镜像构建运行时容器。它的主要优点是将将软件 / 应用程序运行所需的设置和依赖项打包到一个容器中，从而实现了可移植性等优点。

Kubernetes 用于关联和编排在多个主机上运行的容器。

### Kubectl、Kubelet 分别是什么

Kubectl 是一个命令行工具，可以使用该工具控制 Kubernetes 集群管理器，如检查群集资源，创建、删除和更新组件，查看应用程序。

Kubelet 是一个代理服务，它在每个节点上运行，并使从服务器与主服务器通信。

### 简述 Kubernetes 创建一个 Pod 的主要流程？

Kubernetes 中创建一个 Pod 涉及多个组件之间联动，主要流程如下：

- 客户端提交 Pod 的配置信息（可以是 yaml 文件定义的信息）到 kube-apiserver。
- Apiserver 收到指令后，通知给 controller-manager 创建一个资源对象。
- Controller-manager 通过 api-server 将 Pod 的配置信息存储到 etcd 数据中心中。
- Kube-scheduler 检测到 Pod 信息会开始调度预选，会先过滤掉不符合 Pod 资源配置要求的节点，然后开始调度调优，主要是挑选出更适合运行 Pod 的节点，然后将 Pod 的资源配置单发送到 Node 节点上的 kubelet 组件上。
- Kubelet 根据 scheduler 发来的资源配置单运行 Pod，运行成功后，将 Pod 的运行信息返回给 scheduler，scheduler 将返回的 Pod 运行状况的信息存储到 etcd 数据中心。
