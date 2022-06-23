## 架构的演进

软件架构风格从大型机（Mainframe），到[原始分布式](https://icyfenix.cn/architecture/architect-history/primitive-distribution.html)（Distributed），到[大型单体](https://icyfenix.cn/architecture/architect-history/monolithic.html)（Monolithic），到[面向服务](https://icyfenix.cn/architecture/architect-history/soa.html)（Service-Oriented），到[微服务](https://icyfenix.cn/architecture/architect-history/microservices.html)（Microservices），到[服务网格](https://icyfenix.cn/architecture/architect-history/post-microservices.html)（Service Mesh），到[无服务](https://icyfenix.cn/architecture/architect-history/serverless.html)（Serverless）。

以上这些架构设想是基于如何让系统变得更好，而不是如何让系统更加长久的生存下去。而架构演变最重要的驱动力，或者这种“从大到小”趋势的最根本的驱动力，始终都是为了方便某个服务能够顺利地“死去”与“重生”而设计的，个体服务的生死更迭，是关系到整个系统能否可靠续存的关键因素。

系统的更新升级的发展历程可以解释。

而如何让一群不稳定的部件来组合成一个稳定的系统，这就要在整体架构设计有恰当的、自动化的错误熔断、服务淘汰和重建的机制。

