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

## 创建实例Pod和Service
上一笔记我们记录了k8s的安装配置和实例镜像的制作，这节主要介绍在k8s中创建一个实例服务。
- 首先新建一个namespace，namespace.yaml内容如下：
```
apiVersion: v1
kind: Namespace
metadata:
    name: dailyyoga
```
执行命令：kubectl create -f namespace.yaml，这样我们的dailyyoga namespace就创建好了，以后的实例我们都创建在namespace下，避免和k8s默认的namespace混淆不好管理。  
执行命令：kubectl get namespace查看所有的namespace，可以看到我们新建的dailyyoga。当然也可以直接使用命令：kubectl create namespace dailyyoga创建而不是使用文件。
- 创建Pod，go-example.yaml内容如下：
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
这里我们使用namespace配置直接指定Pod创建在dailyyoga下，镜像使用的是上一章节里制作的markbest/go-example:v1，执行命令：kubectl create -f go-example.yaml这样子就创建好了我们的pod。   
如果直接使用kubectl get pods是看不到我们刚才创建的pod的，因为k8s默认的namespace是default，而我们创建的pod在dailyyoga中，所有需要切换当前的namespace到dailyyoga。直接编辑命令行配置文件：vi ~/.zshr，添加以下内容：
```
alias kcd="kubectl config set-context $(kubectl config current-context) --namespace"
``` 
保存文件，然后使用命令：kcd dailyyoga，这样子就切换namespace到dailyyoga，然后再执行kubectl get pods就可以看到以go-example开头的就是我们刚才创建的pod。  
这时候我们命令：curl http://127.0.0.1:8001 是无法访问我们的服务的，应为当前只是在pod中映射了8001端口，没有映射本地的8001端口，所以是调用不通的。可以使用命令：kubectl port-forward go-exmaple-xxx 8001:8001，强制将pod中的8001端口映射到本地的8001端口，此时我们重新执行curl就可以看到访问服务返回的结果，这样子证明我们的服务已经创建成功了。但是在实际生产环境中我们不可能使用port-forward来强制映射端口，此时我们需要创建一个service暴露pod的8001端口到外网，然后外网就能正常访问。
- 创建service，service.yaml内容如下：
```
apiVersion: v1
kind: Service
metadata:
    name: go-example
    namespace: dailyyoga
    labels:
        k8s-app: go-example
spec:
    type: LoadBalancer
    selector:
        k8s-app: go-example
    ports:
        - name: tcp-8001
          protocol: TCP
          port: 8001
          targetPort: 8001
```
LoadBalancer字段表示当前服务类型是负载均衡；selector表示匹配的标签；port和targetPort表示需要映射的端口，yaml文件中的各个字段内容以及具体含义可以通过命令：kubectl explain services和kubectl explain services.spec.type等等来了解。  
执行命令：kubectl create -f service.yaml，我们的service就创建成功了，通过kubectl get services可以看到go-example就是我们创建的service，这时候通过curl就可以访问我们的服务，证明我们创建的service已经成功生效了。  
至此实例的最基础功能已经创建完成了，一个pod和一个service，实际生产环境中会比这个更加复杂，比如使用RS、RC、Deployment来管理服务的动作伸缩和分版本发布，下一节我们继续了解。
