---
author: Sam.Lu
date: 2017-05-13 10:15:10
layout: post
title: 基于官方例程的快速上手笔记
description: 笔记也是备忘:)
tags: [ROS]
categories: [guide]
image:
    feature: rosNamespaces2.png
    credit: ROS
    creditlink: 
---
# ROS学习笔记

## 安装ROS

安装完ros-indigo-desktop-full之后，还要初始化rosdep，这个就是ros dependence，方便以后安装ROS的系统支持库
然后就是环境设置，将ROS环境变量添加到每一次启动终端时自动加载
安装rosinstall，为了以后方便安装各种ros的功能包

## 验证下ROS的安装
```
$ printenv | grep ROS
```
我这里的终端输出：
```
ROS_ROOT=/opt/ros/indigo/share/ros
ROS_PACKAGE_PATH=/opt/ros/indigo/share:/opt/ros/indigo/stacks
ROS_MASTER_URI=http://localhost:11311
ROSLISP_PACKAGE_DIRECTORIES=
ROS_DISTRO=indigo
ROS_ETC_DIR=/opt/ros/indigo/etc/ros
```
## 设置一个ROS工作文件夹
```
$ mkdir -p ~/catkin_ws/src
$ cd ~/catkin_ws/src
```
然后编译一下，以后每次编译都要回到**catkin_ws**路径下：
```
$ cd ~/catkin_ws/
$ catkin_make
```
**catkin_make**会新建俩文件夹："build"和"devel"，在"devel"文件夹下有一些sh文件，运行任意一个，都会让这个"devel"所在的上层**catkin_ws**文件夹变为环境中的最优先路径，我们来搞一下：
```
source devel/setup.bash
```
我们看看现在谁排第一啦：
```
echo $ROS_PACKAGE_PATH
/home/lujun/catkin_ws/src:/opt/ros/indigo/share:/opt/ros/indigo/stacks
```

## 安装版本对应的例程

**packages**是ROS代码组织的一个单位，每个package可以包括库、可执行文件、脚本或其他工件
**manifests**是package的描述，包含定义依赖关系、包的信息如：版本、维护者、协议等
 
## 文件系统工具介绍
rospack find

roscd可以直接进入ros的package里面，比如roscd roscpp，直接就去roscpp的安装目录了，也可以去package的子目录，当然，roscd起作用的地方都在ROS_PACKAGE_PATH里了，

rosls可以查看包里面包含的文件和文件夹

## 新建一个package

先去工作文件夹的src文件夹里
```
catkin_create_pkg beginner_tutorials std_msgs rospy roscpp
```
查看包的一级依赖：rospack depends1 rospy

递归查看包依赖：rospack depends beginner_tutorials

## 自定义自己的程序包
各种修改package.xml


## 编译ROS程序包
开始搞ROS会有很多终端窗口，输入以下命令，惊喜带给你：
```
sudo apt-get install terminator
```
新增标签页：Ctrl + Shift + T
关闭标签页：Ctrl + Shift + W
左右分屏：Ctrl + Shift + E
上下分屏：Ctrl + Shift + O
复制：Ctrl + Shift + C
粘贴：Ctrl + Shift + V
清屏幕：Ctrl + Shift + G
## ROS节点
先roscore开后台
然后各种rosrun
可以用rosnode各种查看，比如rosnode list看看都谁在roscore后台运行着

## ROS topic
rostopic echo [topic]可以看后台的topic

图形化看关系的：
```
rosrun rqt_graph rqt_graph
```
##手动发布消息
发布一次执行消息
```
rostopic pub -1 /turtle1/cmd_vel geometry_msgs/Twist -- '[2.0, 0.0, 0.0]' '[0.0, 0.0, 1.8]'
```
 
按一定频率发送消息：
```
rostopic pub /turtle1/cmd_vel geometry_msgs/Twist -r 1 -- '[2.0, 0.0, 0.0]' '[0.0, 0.0, 1.8]'
```

 -r 1就是1Hz循环发送

图形化看topic里面的信息：

```
rosrun rqt_plot rqt_plot
```

# ROS服务和参数

服务是node之间的另一种通讯方式，基于C/S架构，服务可以给节点发送请求并获得一个响应
rosservice call xxx来调用带参数的服务

rosparam能够操作ROS参数服务器上的数据
rosparam dump 文件名，可以把参数写入文件中
rosparam load 文件名，可以加载参数

## rqt_console和roslaunch

roslaunch 有一个mimic node模式，设置好input和output后，对应的output将模仿input节点的动作

## 创建ROS消息和ROS服务

手动新建msg文件夹里面的msg文件后，需要去package.xml和CMakeLists.txt中添加和打开一堆设置项

服务格式是被---分割，上面是输入，下面是输出
新建一个srv文件夹，然后里面新建或roscp进来一个现成的srv，然后CMakeLists.txt中再各种打开编译项

最后退到catkin_ws，执行catkin_make，就会把.msg和.srv文件转化为ROS支持语言的源代码，比如C++和Python

生成的C++头文件将会放置在~/catkin_ws/devel/include/beginner_tutorials/。 Python脚本语言会在 ~/catkin_ws/devel/lib/python2.7/dist-packages/beginner_tutorials/msg 目录下创建。 lisp文件会出现在 ~/catkin_ws/devel/share/common-lisp/ros/beginner_tutorials/msg/ 路径下.

## 写一个简单的talker和listener

我还是用python吧，只是要注意，用python写完后，还是需要catkin_make下，为了构建消息和服务

代码可以直接下载，然后运行后会看到一个在发发发，一个在收收收，发的停了，收的马上也停了

## 写一个简单的service和client
编写服务节点：求俩数的和
请确保已经在之前导入了AddTwoInts.srv 
然后写两个python，一个是服务端，等着来数做相加，另一个是客户端，发送俩数给服务端

这里有一个细节，客户端不做运算，运算在服务端做，而客户端通过设置服务代理：ServiceProxy来调用"add_two_ints"服务，当然还设置了服务的类型是"AddTwoInts"，以上都还不是细节哦，真正的细节是：
```
add_two_ints = rospy.ServiceProxy('add_two_ints', AddTwoInts)
resp1 = add_two_ints(x, y)
```
这个handle "add_two_ints"可以像函数那样调用，这就高科技了，对于客户端来讲，通过设置代理服务的handler，就可以将整个服务抽象成一个函数啦。
