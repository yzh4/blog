## 持续集成

### 持续集成的概念

 持续集成，Continuous integration ，简称CI。

 随着软件开发复杂度的不断提高，团队开发成员间如何更好地协同工作以确保软件开发的质量已经慢慢成为开发过程中不可回避的问题。尤其是近些年来，敏捷（Agile） 在软件工程领域越来越红火，如何能再不断变化的需求中快速适应和保证软件的质量也显得尤其的重要。

 持续集成正是针对这一类问题的一种软件开发实践。它倡导团队开发成员必须经常集成他们的工作，甚至每天都可能发生多次集成。而每次的集成都是通过自动化的构建来验证，包括自动编译、发布和测试，从而尽快地发现集成错误，让团队能够更快的开发内聚的软件。

 以我经过的项目（假设为A项目）为例进行描述。

 首先，解释下集成。我们所有项目的代码都是托管在SVN服务器上。每个项目都要有若干个单元测试，并有一个所谓集成测试。所谓集成测试就是把所有的单元测试跑一遍以及其它一些能自动完成的测试。只有在本地电脑上通过了集成测试的代码才能上传到SVN服务器上，保证上传的代码没有问题。所以，集成指集成测试。

 再说持续。不言而喻，就是指长期的对项目代码进行集成测试。既然是长期，那肯定是自动执行的，否则，人工执行则没有保证，而且耗人力。对此，我们有一台服务器，它会定期的从SVN中检出代码，并编译，然后跑集成测试。每次集成测试结果都会记录在案。完成这方面工作的就是下面要介绍的Jenkins软件。当然，它的功能远不止这些。在我们的项目中，执行这个工作的周期是1天。也就是，服务器每1天都会准时地对SVN服务器上的最新代码自动进行一次集成测试。

###  持续集成的特点

 它是一个自动化的周期性的集成测试过程，从检出代码、编译构建、运行测试、结果记录、测试统计等都是自动完成的，无需人工干预； 需要有专门的集成服务器来执行集成构建，需要有代码托管工具支持

### 持续集成的作用

 保证团队开发人员提交代码的质量，减轻了软件发布时的压力；

 持续集成中的任何一个环节都是自动完成的，无需太多的人工干预，有利于减少重复过程以节省时间、费用和工作量；

 上面我们了解了持续集成的知识。既然有这么多的好处，那我们怎么样实现它呢？这就是接下来要介绍的名角：Jenkins软件。

## Jenkins

### 特点

易安装：仅仅一个 java -jar jenkins.war，从官网下载该文件后，直接运行，无需额外的安装，更无需安装数据库；

易配置：提供友好的GUI配置界面；

变更支持：Jenkins能从代码仓库（Subversion/CVS）中获取并产生代码更新列表并输出到编译输出信息中；

支持永久链接：用户是通过web来访问Jenkins的，而这些web页面的链接地址都是永久链接地址，因此，你可以在各种文档中直接使用该链接；

集成E-Mail/RSS/IM：当完成一次集成时，可通过这些工具实时告诉你集成结果（据我所知，构建一次集成需要花费一定时间，有了这个功能，你就可以在等待结果过程中，干别的事情）；

JUnit/TestNG测试报告：也就是用以图表等形式提供详细的测试报表功能；

支持分布式构建：Jenkins可以把集成构建等工作分发到多台计算机中完成；

文件指纹信息：Jenkins会保存哪次集成构建产生了哪些jars文件，哪一次集成构建使用了哪个版本的jars文件等构建记录；

支持第三方插件：使得 Jenkins 变得越来越强大；

### 其他集成工具

其它比较著名的持续集成工具有：CruiseControl，TeamCity，Continuum等。