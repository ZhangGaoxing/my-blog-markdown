_原文：[User Stories](http://www.extremeprogramming.org/rules/userstories.html)_

用户故事（User Stories）的用途与用例（Use Case）类似，但不完全相同。它常常被用来估计软件计划发布的时间，同样也被用来代替概括性的需求文档。用户故事是由客户编写的，来描述系统需要为他们做的事情。它们与使用场景（Usage Scenarios <sup>[1]</sup>）相似，除了它们不限于描述用户界面。用户故事使用三句式格式 <sup>[2]</sup>，使用客户术语而非技术术语编写。

用户故事也驱动了验收测试（Acceptance Test）的创建。必须创建一个或多个自动验收测试，以验证用户故事是否已正确实现。

用户故事最大的误解之一是它们如何与传统的需求文档不同。最大的区别是细节层面上的差异。用户故事应该只提供足够的细节，以便对实现故事所需要的时间做出合理的低风险的估计。当需要实现这个故事的时候，开发人员要与客户进行面对面的交流，来获取需求的详细描述。

开发人员估计故事可能需要多长时间才能实现。每个故事都会在“理想的开发时间”中得到1-3周的估计。理想的开发时间是，如果没有干扰，没有其他任务，并且开发人员完全知道该做什么。时间超过三周意味着需要进一步的分解故事。时间小于一周则需要增加细节，结合一些其他故事。用户故事大约80个，上下浮动20个，是创建发布计划的完美数字。

故事和需求文档之间的另一个区别是用户需求的关注度。用户故事应该尽量避免像特定技术、数据库布局、算法等细节，而应该关注用户的需求和利益。

## 注释
1. 描述一个人或多个人或组织如何与系统进行交互的场景。

    A usage scenario, or scenario for short, describes a real-world example of how one or more people or organizations interact with a system. They describe the steps, events, and/or actions which occur during the interaction. Usage scenarios can be very detailed, indicating exactly how someone works with the user interface, or reasonably high-level describing the critical business actions but not the indicating how they're performed. —— [Usage Scenarios: An Agile Introduction](http://www.agilemodeling.com/artifacts/usageScenario.htm)  

2. 建议的三句式格式如下：
    > As a (role) I want (something) so that (benefit).
    
    例如：
    > As a Student I want to purchase a parking pass so that I can drive to school.

    更多参考：[User Stories: An Agile Introduction](http://www.agilemodeling.com/artifacts/userStory.htm)  