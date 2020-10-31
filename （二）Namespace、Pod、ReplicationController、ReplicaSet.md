## 基础概念
- Namespace
可以认为namespaces是kubernetes集群中的虚拟化集群。在一个Kubernetes集群中可以拥有多个命名空间，它们在逻辑上彼此隔离。 他们可以为您和您的团队提供组织，安全甚至性能方面的帮助！
- Pod
Pod是Kubernetes创建或部署的最小/最简单的基本单位，一个Pod代表集群上正在运行的一个进程。
一个Pod封装一个应用容器（也可以有多个容器），存储资源、一个独立的网络IP以及管理控制容器运行方式的策略选项。Pod代表部署的一个单位：Kubernetes中单个应用的实例，它可能由单个容器或多个容器共享组成的资源。
- ReplicationController
ReplicationController会替换由于某些原因而被删除或终止的pod，例如在节点故障或中断节点维护（例如内核升级）的情况下。因此，即使应用只需要一个pod，我们也建议使用ReplicationController。
- ReplicaSet
ReplicaSet（RS）是Replication Controller（RC）的升级版本。ReplicaSet 和  Replication Controller之间的唯一区别是对选择器的支持。ReplicaSet支持labels user guide中描述的set-based选择器要求， 而Replication Controller仅支持equality-based的选择器要求。

## 如何使用
上一笔记我们记录了k8s的安装配置和实例镜像的制作，这节主要介绍在k8s中创建一个实例服务。
- 首先新建一个namespace，namespace.yaml内容如下：
```
apiVersion: v1
kind: Namespace
metadata:
    name: dailyyoga
```
执行命令：kubectl create -f namespace.yaml，这样我们的dailyyoga namespace就创建好了，以后的实例我们都创建在namespace下，避免和k8s默认的namespace混淆不好管理.执行命令：kubectl get namespace查看所有的namespace，可以看到我们新建的dailyyoga。当然也可以直接使用命令：kubectl create namespace dailyyoga创建而不是使用文件。
- 创建我们的实例，go-example.yaml内容如下：
```
apiVersion: v1
kind: Pod
metadata:
    name: go-example
    namespace: dailyyoga
    labels:
        k8s-app: go-example
spec:
    containers:
        - image: markbest/go-example:v1
          name: go-example
```
这里我们使用namespace直接指定实例创建在dailyyoga下，镜像使用的是上一章节里制作的markbest/go-example:v1，执行命令：kubectl create -f go-example.yaml这样子就创建好了我们的pod go-example.

