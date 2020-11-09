---
title: ROS
typora-root-url: ../../source
typora-copy-images-to: ../images/ROS_learning.assets
mathjax: true
categories: Computer Vision
tags:
- Robotics
---

#### 关于ROS的一些笔记

**图概念概述**

<!--more-->

<img src="/images/ROS_learning.assets/image-20201105192111214.png" alt="image-20201105192111214" style="zoom: 25%;" />

- Nodes:节点,一个节点即为一个可执行文件，它可以通过ROS与其它节点进行通信。
- Messages:消息，消息是一种ROS数据类型，用于订阅或发布到一个话题。
- Topics:话题,节点可以发布消息到话题，也可以订阅话题以接收消息。
- Master:节点管理器，ROS名称服务 (比如帮助节点找到彼此)。

<img src="/images/ROS_learning.assets/image-20201105215222061.png" alt="image-20201105215222061" style="zoom:25%;" />

**节点**

- 一个节点其实只不过是ROS程序包中的一个可执行文件。
- ROS节点可以使用ROS客户库与其他节点通信。
- 节点可以发布或接收一个话题。
- 节点也可以提供或使用某种服务。

**ROS Topics**

- turtlesim_node节点和turtle_teleop_key节点之间是通过一个ROS话题来互相通信的。
- turtle_teleop_key在一个话题上发布按键输入消息，而turtlesim则订阅该话题以接收该消息。

**ROS Messages**

- 话题之间的通信是通过在节点之间发送ROS消息实现的。
- 对于发布器(turtle_teleop_key和订阅器(turtulesim_node)之间的通信，发布器和订阅器之间必须发送和接收相同类型的消息。
- 这意味着话题的类型是由发布在它上面的消息类型决定的。
- 使用rostopic type命令可以查看发布在某个话题上的消息类型。

<img src="/images/ROS_learning.assets/image-20201105215111887.png" alt="image-20201105215111887" style="zoom:25%;" />

**ROS Services**

- 服务（services）是节点之间通讯的另一种方式。
- 服务允许节点发送请求（request） 并获得一个响应（response）

消息(msg)和服务(srv)