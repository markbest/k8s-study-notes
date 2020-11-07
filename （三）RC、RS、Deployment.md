## 基础概念
- ReplicationController  
ReplicationController会替换由于某些原因而被删除或终止的pod，例如在节点故障或中断节点维护（例如内核升级）的情况下。因此即使应用只需要一个pod，我们也建议使用ReplicationController。
- ReplicaSet  
ReplicaSet（RS）是Replication Controller（RC）的升级版本。ReplicaSet和Replication Controller之间的唯一区别是对选择器的支持。ReplicaSet支持labels user guide中描述的set-based选择器要求，而Replication Controller仅支持equality-based的选择器要求。