---
layout: post
title: ROS和MATLAB实现服务通信
date: 2022-07-06 09:56:00-0400
description: Shown how to use MATLAB to achieve ROS node communication
tags: ROS
categories: tech
giscus_comments: true
related_posts: false
featured: true
---
​
- 硬件条件：两台装有ubuntu的电脑
- 需求：一台工控机向机器人电机端发送指令，一台电脑安装有MATLAB进行实施运动规划。MATLAB和工控机上的ROS节点建立服务通信，采用自定义的数据类型方便数据传输
- 实现方式：ROS中的服务通信，工控机是server端，laptop是client段，服务通信要求client发送数据并从server端接收返回值
- 前置工作：网线直连，IP地址配置

> 参考链接：
> 
> [ROS多机器通信_awhuter的博客](https://blog.csdn.net/qq_41253960/article/details/121303505)

## 一、自定义服务数据类型的实现

> 参考链接：
> 
> [Create Custom Messages from ROS Package- MATLAB & Simulink- MathWorks 中国](https://ww2.mathworks.cn/help/ros/ug/create-custom-messages-from-ros-package.html?searchHighlight=rosgenmsg&s_tid=srchtitle_rosgenmsg_2)
> 
> [ROS Custom Message Support- MATLAB & Simulink- MathWorks 中国](https://ww2.mathworks.cn/help/ros/ug/ros-custom-message-support.html#butuk98)

后者指明了必须先定义好的文件结构（每一个条目都不能少）

 其中“custom_robot_msgs”是数据包的名字，本文中制定的数据包名为“lander”。上述文件夹都被放在一个package文件夹内，在该目录下执行：
```bash
rosgenmsg('......\package'')
```
即可在数据包lander的同级目录下生成“matlab_msg_gen_ros1”文件。这行代码没有抱错的话，按照提升继续完成后续操作即可。

执行rosmsg list可以查看自定义通信数据结构是否被顺利建立。会生成两个新的数据类型，lander/...Request和lander/...Response

## 二、MATLAB端代码编写
工控机端采用的语言是C++，编译器是vsCode，其环境配置和服务端实现可供参考的资料较多。

> 参考链接：
> 
> [Introduction · Autolabor-ROS机器人入门课程《ROS理论与实践》零基础教程](http://www.autolabor.com.cn/book/ROSTutorials/)

MATLAB端服务通信可供参考资料较少，主要参考MathWorks Help Center。

主要代码如下：
```matlab
% 初始化ROS网络, 网线通讯已经搭建好，不需要再设置默认端口
rosinit 
%% 使用'rossvcclient'创建一个客户端，采用我们自定义的数据结构
% rossvcclient的参数是在服务器（lander）端定义的话题名称
lander_client = rossvcclient('PlanMsg');
% 检查通讯是否建立
if(isServerAvailable(lander_client))
    [connectionStatus,connectionStatustext] = waitForServer(lander_client)
end
% 创建服务请求函数
req_msg = rosmessage(lander_client);
% The default service request message is an empty message of type serviceclient.ServiceType.
req_msg.CommandIndex = 0;
% 向工控机发送命令
lander_resp = call(lander_client, req_msg);
```
在matlab端获取ROS全局参数【记得要在参数名前加/】：

```matlab
% 启动ROS参数服务器
GlobalParam = rosparam;
% 判断上个指令的动作是否执行完毕，并发送数据
    if count ~= 0
        while isFinishFlag == false
            isFinishFlag = get(GlobalParam, '/isFinishFlag');
        end
    end
```