## 架构的演进

软件架构风格从大型机（Mainframe），到[原始分布式](https://icyfenix.cn/architecture/architect-history/primitive-distribution.html)（Distributed），到[大型单体](https://icyfenix.cn/architecture/architect-history/monolithic.html)（Monolithic），到[面向服务](https://icyfenix.cn/architecture/architect-history/soa.html)（Service-Oriented），到[微服务](https://icyfenix.cn/architecture/architect-history/microservices.html)（Microservices），到[服务网格](https://icyfenix.cn/architecture/architect-history/post-microservices.html)（Service Mesh），到[无服务](https://icyfenix.cn/architecture/architect-history/serverless.html)（Serverless）。

以上这些架构设想是基于如何让系统变得更好，而不是如何让系统更加长久的生存下去。而架构演变最重要的驱动力，或者这种“从大到小”趋势的最根本的驱动力，始终都是为了方便某个服务能够顺利地“死去”与“重生”而设计的，个体服务的生死更迭，是关系到整个系统能否可靠续存的关键因素。

系统的更新升级的发展历程可以解释。

而如何让一群不稳定的部件来组合成一个稳定的系统，这就要在整体架构设计有恰当的、自动化的错误熔断、服务淘汰和重建的机制。

## 前端工程

前后端分离式项目，只要前后端相互约定好服务的位置及模型即可。前端可以使用Mock.js来拦截所有的远程服务请求，并以事先准备好的数据来完成对这些请求的响应。纯前端方式运行的时候，所有对数据的修改请求都是无效的，比如用户注册，无论输入什么用户名，密码，由于请求的响应是静态预设好的，所以最终都会以同一个预设的用户登录。只维护在前端的状态依然是可以变动的，比如购物车、收藏夹的增删改。让后端服务保持无服务，而把状态维持在前端中的设计，对服务的伸缩性和系统的鲁棒性都有着极大的益处，多数情况下都是值得提倡的良好设计。而其伴随而来的状态数据导致请求头变大，链路安全性等问题。

