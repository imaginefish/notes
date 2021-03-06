# Maven 学习笔记
## 搭建 Maven 私服
>私服是一种私有服务器,是在局域网中搭建的一种特殊的远程仓库,目的是代理远程仓库及部署第三方构建，私服搭建成功之后，当maven需要下载构件时，直接请求私服。私服上存在则下载到本地仓库，不存在才请求外部的远程仓库，将构件下载到私服，再提供给本地仓库下载，可以减少重复网络流量下载问题。

目前搭建 maven 私服主要有以下三种方式，最主流的就是 Nexus。
- Nexus 搭建 maven 私服
- Artifactory 搭建 maven 私服
- Apache Ar-chiva 搭建 maven 私服（不常用）

### Nexus介绍
>Nexus 是一个强大的 Maven 仓库管理器，它极大地简化了本地内部仓库的维护和外部仓库的访问。如果使用了公共的 Maven 仓库服务器，可以从 Maven 中央仓库下载所需要的构件（Artifact），但这通常不是一个好的做法。正常做法是在本地架设一个 Maven 仓库服务器，即利用 Nexus 私服可以只在一个地方就能够完全控制访问和部署在你所维护仓库中的每个 Artifact。Nexus 在代理远程仓库的同时维护本地仓库，以降低中央仓库的负荷,节省外网带宽和时间，Nexus 私服就可以满足这样的需要。Nexus 是一套“开箱即用”的系统不需要数据库，它使用文件系统加 Lucene 来组织数据。 Nexus 使用 ExtJS 来开发界面，利用 Restlet 来提供完整的 REST APIs，通过 m2eclipse 与 Eclipse 集成使用。 Nexus 支持 WebDAV 与 LDAP 安全身份认证。 Nexus 还提供了强大的仓库管理功能，构件搜索功能，它基于 REST，友好的 UI 是一个 extjs 的 REST 客户端，它占用较少的内存，基于简单文件系统而非数据库。

在本地构建 nexus 私服的好处有：
- 加速构建；
- 节省带宽；
- 节省中央 maven 仓库的带宽；
- 稳定（应付一旦中央服务器出问题的情况）；
- 控制和审计；
- 能够部署第三方构件；
- 可以建立本地内部仓库；
- 可以建立公共仓库

这些优点使得 Nexus 日趋成为最流行的 Maven 仓库管理器。

参考来源：[https://www.cnblogs.com/wuzhenzhao/p/13307444.html](https://www.cnblogs.com/wuzhenzhao/p/13307444.html)