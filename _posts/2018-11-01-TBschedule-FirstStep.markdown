---
layout:     post
title:      "Zookeeper与TBSchedule"
date:       2018-11-01
author:     "yzgu"
header-img: "img/post-bg-20181002.png"
tags:
    - TBSchedule
    - Zookeeper
---


## 所有工作的第一条：先让他跑起来

1 tomcat运行起来后无法启动调度平台，加载zk配置文件频繁报错

调整启动调度管理配置，如果只是做系统管理，应该设置为false，如果需要执行调度任务，应该置为true
![img](/img/zookeeper/9.png)
 

在ConsoleManager.initial方法中调整
![img](/img/zookeeper/10.png)

 

改为手动loadConfig获得默认设置

2 zk报错，跳转到配置页面
![img](/img/zookeeper/11.png)

 

此时无法进行任何操作，修复zk即可，查看页面源代码，进入管理主页：
http://localhost:8080/TBSchedule/schedule/index.jsp?manager=true

3 若zk配置正确，直接进入http://localhost:8080/TBSchedule/schedule/index.jsp页面
![img](/img/zookeeper/12.png)

无法进行配置操作，此时zk没有数据，也无法执行管理和调度作业，根据2中的情况，可以直接进入管理主页地址：http://localhost:8080/TBSchedule/schedule/index.jsp?manager=true

4 在页面进行配置调度，先进行调度策略配置，再进行任务管理配置，机器管理能找到自己当前所在主机即可，处理线程组列表等不用加以考虑。
![img](/img/zookeeper/13.png)

配置调度：
 
策略名称：按规则随便配置
任务类型：阅读完代码后决定配置方式
任务名称：描述不清楚，阅读完代码决定配置方式
任务参数：不配置
单JVM最大线程组数量：不配置或配置为1
最大线程组数量：不配置或配置为1
IP地址：不配置，保持默认参数

任务管理配置：
任务名称：需要和调度策略配置中的任务名称一致
任务处理的SpringBean：阅读完代码后进行配置
心跳频率：保持默认
假定服务死亡间隔：保持默认
线程数：保持默认
处理模式：SLEEP
每次获取数据量：保持默认
每次执行数量：保持默认
没有数据时休眠时长：保持默认
每次处理完数据后休眠时间：保持默认
执行开始时间：按照提示配置0 * * * * ?
执行结束时间：保持默认
单线程组最大任务项：保持默认
自定义参数：保持默认
任务项：按照提示配置0
 
![img](/img/zookeeper/14.png)

5 配置完调度策略和任务管理配置后，查看ZK数据结构
 
![img](/img/zookeeper/15.png)

了解任务调度配置，数据结构如下：
/taobao/spectator/
				baseTaskType/
							mytaskTest/mytaskTest/
												server 调度管理器
												taskItem	任务项

				factory
					主机节点
				strategy
						taskTest
							执行taskTest任务的主机节点
6 配置完后，任务执行仍不成功，报空指针异常
 
![img](/img/zookeeper/16.png)

![img](/img/zookeeper/17.png)
 
发现是applicationcontext为空，spring容器加载时没有加载到Spring上下文环境
 
![img](/img/zookeeper/18.png)

调整web.xml文件，新增spring容器配置文件application-context.xml文件，这里命名为schedule.xml文件

Web.xml中增加配置加载Spring容器的Listener，并指定spring上下文环境使用的配置文件

 
![img](/img/zookeeper/19.png)

在新增的schedule.xml文件中添加配置

 
![img](/img/zookeeper/20.png)

7 添加相应配置后，applicationcontext仍报空指针，spring容器需要将对应的FactoryBean先注册到容器中，并在它调用的bean之前定义，同时将applicationcontext设置为静态变量

8 报找不到com.taobao.pamirs.schedule.DemoTaskBean

将页面任务配置中的
 
![img](/img/zookeeper/21.png)
 
调整为
 
![img](/img/zookeeper/22.png)

和schedule.xml配置文件中的测试Bean-id保持一致