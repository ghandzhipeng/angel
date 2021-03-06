# Louvain(FastUnfolding)

> Louvain(FastUnfolding)算法是经典的社区发现算法, 通过优化[模块度](https://en.wikipedia.org/wiki/Modularity)指标来达到社区划分的目的。

## 1. 算法介绍
Louvain算法包含两个过程
 - 模块度优化
 - 社区折叠
我们通过两个ps向量来维护节点的社区id以及社区id对应的权重信息。Spark端每个worker维护一部分节点和对应的邻接信息，包括节点的邻居以及对应的连边权重。
- 在模块度优化阶段，每个worker根据模块度变化量计算自己维护节点的新的社区归属。社区归属的更新以batch的形式实时更新到ps。
- 在社区折叠阶段，我们根据当前的社区归属情况，构造新的网络，其中新的网络节点对应与折叠前网络的社区，新的连边对应于折叠前网络社团直接节点的连边权重之和。在开始下一阶段的模块度优化之前，我们需要校正社区id，使得每个社区的id标识为社区内某个节点的id。 这里我们使用社区中所有节点id最小的作为标识。

## 2. 运行

### 参数

- input： hdfs路径，输入网络数据，每行两个长整形id表示的节点（如果是带权网络，第三个float表示权重），以空白符或者逗号分隔，表示一条边
- output： hdfs路径， 输出节点对应的社区归属， 每行一条数据，表示节点对应的社区id值，以tap符分割
- numFold： 折叠次数
- numOpt：每轮模块度优化次数
- eps：模块度增量下限
- batchSize：节点更新batch的大小
- partitionNum： 输入数据分区数
- isWeighted：是否带权
