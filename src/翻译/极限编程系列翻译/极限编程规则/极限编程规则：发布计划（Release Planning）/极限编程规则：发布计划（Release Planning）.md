_原文：[Release Planning](http://www.extremeprogramming.org/rules/planninggame.html)_

发布计划会议用来创建发布计划。发布计划要列出整个项目，用于为每个迭代创建迭代计划。

对于技术人员来说做出技术决策，业务人员来说做出业务决策是很重要的。发布计划有一套规则，允许每个参与项目的人做出他们自己的决定。规则定义了一种方法，允许每个人去协商时间表。

发布计划会议的实质是让开发团队评估每个用户故事需要花费多少“理想的开发时间”。理想的开发时间是，如果没有干扰，没有其他任务，并且开发人员完全知道该做什么，其中还要包括测试。客户决定什么故事是最重要的或要以最高优先级完成。

用户故事被打印或写在卡片上。开发人员和客户一起在一张大的桌子上移动卡片，来创建一组需要在第一个（或下一个）版本中实现的故事。一个有用的，可测试的系统，要有尽快交付的良好的商业意识。

发布计划可以按照时间或范围进行规划。项目开发速度用于确定在给定日期之前可以实现多少个故事（时间），或者确定一组故事需要多长时间才能完成（范围）。在按时间进行规划时，将迭代次数乘以项目开发速度，以确定可以完成多少个用户故事。当按照范围进行规划时，将估计的实现用户故事的总周数除以项目速度，以确定在发布准备好之前有多少次迭代。

每一次迭代计划都是在每次迭代开始之前，而不是预先计划好的。发布计划会议被称为计划游戏（Planning Game），规则可以在 [Portland Pattern Repository](http://wiki.c2.com/?PlanningGame) 中找到。

当最终的发布计划被创建，并且管理层对其不满时，不能只更改对开发时间的估计。时间的估计必须是有效的，并且在迭代计划会议时也是必需的。现在对时间的低估会在项目后期出现问题。相反的，应去谈判一个可接受的发布计划，直到开发人员、客户和管理层都同意发布计划。

发布计划的基本哲学原理是，项目可以通过四个变量进行量化：范围、资源、时间和质量。范围是要完成多少功能；资源是有多少人可用；时间是项目什么时候完成；质量就是软件的好坏以及测试的好坏。

没有人能控制所有四个变量。当无意中改变一个变量时，会引起其他变量的改变。请注意，将质量降低到不太出色的程度都会对其他三个变量产生不可预见的影响，而这些影响可能直到项目在最后期限之前放慢速度时才会变得明显。

本质上，只有前三个变量是可以改变的。管理人员可以设置其中的两个变量，而第三个变量将由开发团队设置。还可以让开发人员通过一次招聘过多的人来缓和客户希望立即完成项目的愿望。