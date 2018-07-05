Kubernetes正在迅速成为在云中部署和管理软件的新标准。然而，Kubernetes提
供的所有力量，都带来了一个陡峭的学习曲线。作为一个新来者，试图解析官方文档可能太过困难。有许多不同的组成部分组成了官方文档系统，
并且很难判断哪些会与你的使用相关。这篇博客文章将提供Kubernetes的简化视图，
但它将尝试对最重要的组件以及它们如何组合在一起进行高层次的概述。
原文链接：https://medium.com/google-cloud/kubernetes-101-pods-nodes-containers-and-clusters-c1509e409e16

首先，让我们看看硬件是如何表示的
## Hardware
### Nodes（节点）
![1](https://github.com/rushoo/kubernetes-/blob/master/Nodes.png)
在Kubernetes，一个节点是计算硬件的最小单元。它是集群中单个机器的表示。
在大多数生产系统中，节点很可能是数据中心中的物理机器，或者是托管在云提供商上的虚拟机，
比如Google云平台。然而，不要让传统限制你;理论上，你可以用几乎任何东西来做一个节点。
将机器视为“节点”，允许我们插入一层抽象层。现在，我们可以简单地将每台机器看作是一组可以使用的CPU和RAM资源，
而不是担心任何一台机器的独特特性。通过这种方式，任何机器都可以替代Kubernetes集群中的任何其他机器。

### Cluster(集群) 
![2](https://github.com/rushoo/kubernetes-/blob/master/Cluster.png)
尽管使用单个节点是有用的，但它并不是Kubernetes的全部。
一般来说，你应该把集群看作一个整体，而不是过分担心单个节点的状态。
在Kubernetes，节点将它们的资源集中在一起，形成一个更强大的机器。
当您将程序部署到集群上时，集群会智能地为各个节点分配工作。
如果添加或删除任何节点，集群将在必要时转移工作。
对于程序或程序员来说，这并不重要，因为单个机器实际上是在运行代码。
如果这种类似于hivemind的系统让你想起星际迷航中的Borg，那么你并不陌生;“Borg”是Kubernetes的内部谷歌项目的名称。（K8s前身）

### Persistent Volumes 
![3](https://github.com/rushoo/kubernetes-/blob/master/Persistant%20Volume.png)
因为在集群上运行的程序不能保证在特定的节点上运行，所以数据不能保存到文件系统中的任意位置。如果一个程序试图将数据保存到一个文件中，但是随后被重新定位到一个新的节点，那么该文件将不再是程序期望的位置。因此，与每个节点相关联的传统本地存储都被当作临时缓存来保存程序，但是本地保存的任何数据都不能持久。

为了将数据持久化，Kubernetes使用Persistent Volumes。尽管所有节点的CPU和RAM资源都能由集群有效地合用和管理的，但是持久的文件存储却不像那样。相反，本地或云驱动器可以作为持久卷附加到集群上。这可以被认为是将一个外部硬盘插入到集群中。持久卷提供了一个文件系统，该文件系统可以安装到集群中，而不需要与任何特定节点相关联。

## Software
### Containers 
![4](https://github.com/rushoo/kubernetes-/blob/master/Container.png)
运行在Kubernetes上的程序被打包成Linux容器。容器是一个被广泛接受的标准，所以已经有许多预先构建的镜像可以部署在Kubernetes上。
容器化允许您创建自含的Linux执行环境。任何程序及其所有依赖项都可以被打包成一个单独的文件，然后在internet上共享。任何人都可以下载该容器并将其部署到基础设施上，只需很少的设置。创建容器可以通过编程方式完成，从而形成强大的CI和CD管道。
多个程序可以被添加到单个容器中，但是如果可能的话，最好每个容器只创建一个进程。多个小容器的组合通常会比一个大的容器要好。如果每个容器都有一个紧密的焦点，更新就更容易部署，而且问题更容易诊断。

### Pods 
![5](/Pods.png)
与过去你可能使用过的其他系统不同，Kubernetes不直接运行容器;相反，它将一个或多个容器封装到一个称为pod的高级结构中。同一pod中的任何容器都将共享相同的资源和本地网络。容器可以很容易地与相同的容器中的其他容器进行通信，就像它们在同一台机器上一样，同时保持与其他容器的隔离程度。
在Kubernetes，pod被用作复制的基本单位。如果你的应用变得太流行，而单个pod实例无法承载负载，那么可以通过配置Kubernetes在必要时将pod的新副本部署到集群中。即使负载不高，标准情况下，在生产系统的任何时候都保有pod的多个副本，以应对负载平衡和故障。
Pods可以容纳多个容器，但如果可能，最好一个pod一个容器。因为pod是作为一个单位来放大和缩小的，所以一个pod中所有的容器都必须一起伸缩，而不管它们的各自需要。这将导致很大的资源浪费。因此，pods应该保持尽可能小的范围，通常只持有一个主进程和与它紧密耦合的辅助容器（这些辅助容器通常被称为“side-cars”）

### Deployments 
![6](https://github.com/rushoo/kubernetes-/blob/master/Deployment.png)
尽管pods是Kubernetes的基本计算单元，但它们通常不是直接在集群上启动的。相反，pods通常由另一层抽象管理：deployment
一个deployment的主要目的是声明一个pod应该一次运行多少个副本。当一个deployment被添加到集群时，它会自动增加pod到所请求的数量，然后监视它们。如果一个pod死亡，deployment会自动重新创建它。
使用deployment，你不需要手动处理pods。可以声明系统的期望状态，它将自动为你管理。

### Ingress 
![7](https://github.com/rushoo/kubernetes-/blob/master/Ingress.png)
使用上面描述的概念，您可以创建一个节点集群，并将pods部署到集群上。然而，还有最后一个问题需要解决：允许外部流量访问你的应用程序。
在默认情况下，Kubernetes提供了在pod和外部环境之间的隔离。如果想要与运行在一个pod中的服务通信，你必须打开一个通信通道。这被称为ingress。
向集群中添加ingress有多种方法。最常见的方法是添加一个Ingress控制器，或者一个负载平衡器（LoadBalancer）。这两个选项之间的精确权衡超出了这篇文章的范围，但你必须意识到，在能使用Kubernetes之前，ingress是你需要处理的东西。
