# 基础概念
## ReplicationController  
ReplicationController会替换由于某些原因而被删除或终止的pod，例如在节点故障或中断节点维护（例如内核升级）的情况下。因此即使应用只需要一个pod，我们也建议使用ReplicationController。
## ReplicaSet  
ReplicaSet（RS）是Replication Controller（RC）的升级版本。ReplicaSet和Replication Controller之间的唯一区别是对选择器的支持。ReplicaSet支持labels user guide中描述的set-based选择器要求，而Replication Controller仅支持equality-based的选择器要求。
## Deployment
Deployment为Pod和Replica Set提供声明式更新。
你只需要在Deployment中描述您想要的目标状态是什么，Deployment controller就会帮您将Pod和ReplicaSet的实际状态改变到您的目标状态。您可以定义一个全新的Deployment来创建ReplicaSet或者删除已有的Deployment并创建一个新的来替换。

# 实例操作
上一节我们创建了一个Pod和Service已经简单实现了我们的服务，但是只是一个Pod和Service在生产环境中是不够的的，比如如果Pod因为某种情况终止或者误删除这就会导致服务不可用，为了规避这种情况我们可以使用RC、RS、Deployment来对Pod进行实时管理。  
我们先看看怎么创建ReplicationController来优化我们的服务。先删除上一节创建的Pod，用命令：kubectl delete pod go-example-774f998fbf-9l2tq，然后创建go-example-rc.yaml，文件内容如下：
```
apiVersion: apps/v1
kind: ReplicationController
metadata:
    name: go-example
    namespace: dailyyoga
spec:
    replicas: 3
    selector:
        matchLabels:
            app: go-example
    template:
        metadata:
            labels:
                app: go-example
        spec:
            containers:
                - name: go-example
                  image: markbest/go-example:v1
```
- Kind：ReplicationController（表示类型是ReplicationController）
- replicas: 3（表示实例数量是3，也就是总共会创建三个Pod，可以根据实际需求更改实例数量）
- matchLabels（表示匹配的标签，根据这个Pod标签来判断是否满足replicas的数量）  

然后执行kubectl create -f go-example-rc.yaml等待创建完成就可以看到我们创建的三个Pod，然后执行curl命令就可以看到服务确实已经均衡负载到了不同的Pod。  
![](https://github.com/markbest/k8s-study-notes/blob/main/images/go-exmaple-rc-show.png "") 
