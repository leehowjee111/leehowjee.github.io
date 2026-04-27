+++
date = '2026-04-23T18:59:50+08:00'
draft = false
title = 'c++17异步服务器'
slug: cpp17-async-server  # 关键：指定URL后缀，避免用中文/特殊字符
+++
# 总体复盘阶段
### 一 项目的运行流程
###### <span>&emsp;&emsp;</span>整个项目的入口在chat_server.cpp中，通过一个server()函数进行服务器的初始化以及启动。在main函数中通过server()进行调用。接下来对server()进行分析

<span>&emsp;&emsp;</span> 首先是**注册路由**和**创建io操作的实例**（包括epoll等等），有 '/' , '/send' , '/recv'等等,每个路由都代表了一种”动作”吧，例如使用send时，客户端向服务器发送的消息，而recv则是客户端接受了服务器的消息。然后是启动服务器，即 server().do_start();这个函数相当于启动了服务器，把创建套接字，绑定地址，监听网卡，操作accept的队列，这几个操作封装在了一起。当从accept队列中成功连接一个后，会调用connection_handler进行处理后面的信息。详细内容在此不再赘述，后面会对每个模块进行详解。

### 二 项目的整体结构

