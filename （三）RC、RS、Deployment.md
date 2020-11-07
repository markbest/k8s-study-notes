# 基础概念
## ReplicationController  
ReplicationController会替换由于某些原因而被删除或终止的pod，例如在节点故障或中断节点维护（例如内核升级）的情况下。因此即使应用只需要一个pod，我们也建议使用ReplicationController。
## ReplicaSet  
ReplicaSet（RS）是Replication Controller（RC）的升级版本。ReplicaSet和Replication Controller之间的唯一区别是对选择器的支持。ReplicaSet支持labels user guide中描述的set-based选择器要求，而Replication Controller仅支持equality-based的选择器要求。
## Deployment
Deployment为Pod和Replica Set提供声明式更新。你只需要在Deployment中描述您想要的目标状态是什么，Deployment controller就会帮您将Pod和ReplicaSet的实际状态改变到您的目标状态。您可以定义一个全新的Deployment来创建ReplicaSet或者删除已有的Deployment并创建一个新的来替换。

# 实例操作
## 使用RC优化服务
上一节我们创建了一个Pod和Service已经简单实现了我们的服务，但是只是一个Pod和Service在生产环境中是不够的的，比如如果Pod因为某种情况终止或者误删除这就会导致服务不可用，为了规避这种情况我们可以使用RC、RS、Deployment来对Pod进行实时管理。  
我们先创建一个ReplicationController，执行命令`kubectl delete pod`删除上一节创建的Pod，然后创建go-example-rc.yaml，文件内容如下：
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

然后执行`kubectl create -f go-example-rc.yaml`等待创建完成就可以看到我们创建的三个Pod，然后执行curl命令就可以看到服务确实已经均衡负载到了不同的Pod。  
![](https://github.com/markbest/k8s-study-notes/blob/main/images/go-example-rc-show.png "")  
我们测试删除一个Pod，执行`kubectl delete pod go-example-774f998fbf-cjs2j`，等待一会儿既可以看到一个Pod已经重新创建了，这个就是ReplicationController对Pod的管理，始终会保持Pod的数量满足设置的replicas，数量不足会重新创建，这样就一定程度上保证了Pod的可用性。  
ReplicaSet是ReplicationController的升级版本，功能相似、使用方法基本一致，仅仅需要将Kind修改为ReplicaSet即可。两者的区别在于：
- RC的selector只能用等式（比如app=go-example或者app!=go-example）来获取相关的pod。
- RS除了支持等式还支持通过集合的方式，比如app in (go-example,go-example-v1)，使用RS可以让运维进行更复杂的查询。  

官方已经推荐我们使用RS和Deployment来代替RC。  

## 弹性伸缩
鉴于RC或者RS的功能特性，我们当然可以对Pod数量进行弹性伸缩管理，有两种方法：
- 执行命令`kubectl scale rc rc-name --replicas=1`。
- 手动编辑`kubectl edit rc go-example`，然后设置replicas参数然后保存即可。  
## 滚动升级
RC或RS是不支持滚动升级的，如果需要实现滚动升级非常麻烦，往往需要以下操作：
- 手动编辑`kubectl edit rc go-example`，然后修改image参数然后保存。
- 删除之前创建的所有Pod。因为RC只会判断Pod数量是否满足replicas，不关心image的变更，所以我们需要先删除旧的Pod，然后等待RC创建新的Pod这样就实现了滚动升级，虽然能够实现但是操作起来非常麻烦，如果Pod数量非常多，那操作起来能让人奔溃。  

为了更好的实现滚动升级，我们需要进一步使用Deployment来优化我们的服务。创建go-example-dy.yaml，内容如下：
```
apiVersion: apps/v1
kind: Deployment
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
对比创建RC的文件内容，我们可以看到变化不大，就是Kind字段进行了变更。执行`kubectl create -f go-exmaple-dy.yaml`，等待执行完成然后通过命令`kubectl get all`就可以看到创建好的Deployment和RS。
![](https://github.com/markbest/k8s-study-notes/blob/main/images/go-example-dy-show.png "")   
执行`kubectl scale deployment go-exmaple --replicas=1`既可以实现Deployment的弹性伸缩，执行`kubectl set image deployment/go-exmaple image=markbest/go-example:v2`既可以实现动态更换Pod镜像进而实现滚动升级。



