# SpringMVC

ssm：mybatis + Spring + SpringMVC  **MVC三层架构**

## 回顾MVC

- MVC是模型（Model）、视图（View）、控制器（Controller）的简写，是一种软件设计规范。
- 是将业务逻辑、数据、显示分离的方法来组织代码
- MVC主要作用是降低了与业务逻辑间的双向耦合
- MVC不是一种设计模式，MVC是一种架构模式。当然不同的MVC存在差异



**Model：**数据模型，提供要展示的数据，因此包含数据和行为，可以认为是领域模型或JavaBean组件（包含数据和行为），不过现在一般都分离开：Value Object（数据Dao）和服务层（行为Service）。也就是模型提供了模型数据查询和模型数据的状态更新等功能，包括数据和业务。

**View:**负责进行模型的展示，一般就是我们见到的用户界面，客户想要看到的东西。

**Controller:**接收用户请求，委托个模型进行处理（状态改变），处理完毕后把返回的模型数据返回给视图，由视图负责展示。也就是说控制器做了调度员的工作。

![image-20210824094205184](C:\Users\feifeng.yang\AppData\Roaming\Typora\typora-user-images\image-20210824094205184.png)