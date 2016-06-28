---
title: 我对MVVM的理解
date: 2016-06-22 14:23:45
tags: [设计模式]
language: zh-CN

---
# MVC
先说MVC模式，在前端应用开发中MVC是一个最经典最常用的设计模式，也是被Apple封为神的设计模式。我个人项目应用也是最多的一种；在MVC中所有的对象被归类为一个model，一个view，或一个controller。Model持有数据，View显示与用户交互的界面，而View Controller调解Model和View之间的交互。如下图（盗图）：<br/>
![MVC](http://7xoo3c.com1.z0.glb.clouddn.com/blogmvc.jpg "MVC")

图中Controler起链接View和Model的作用，正式由于Controler所担责任的职责过大以至于Controler的任务过于庞大复杂，对问题的调试查找产生很大困难。

# MVVM
MVVM模式广泛应用在WPF项目开发中，使用此模式可以把UI和业务逻辑分离开，使UI设计人员和业务逻辑人员能够分工明确。它去除Controller中的逻辑层新增VM层，合并View和Controller层为View层，大大分担了Controller和View层中的业务逻辑负担。

![MVC](http://7xoo3c.com1.z0.glb.clouddn.com/blog_mvvm.png "MVC")

# 等过段时间理解透了再写！！！
