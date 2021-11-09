---
layout: post
title: MySQL错误“ERROR 2006 （HY000）：MySQL server has gone away”
date: 2021-11-09
Author: Los
tags: [mysql]
comments: false
---

MySQL导入数据库错误“ERROR 2006 （HY000）：MySQL server has gone away” 这是MySQL数据库中常见的一个错误，导致这个错误的原因主要有几个。一般而言，这实际上意味着“您的SQL语句失败，因为？？？失去与数据库的连接”，我们要做的就是检查出？？？是什么。以下是一些常见的情况以及如何检查？？？是什么？

1. MySQL 服务器真的不见了

     我们可以通过检查服务器正常运行时间（uptime）和服务器的错误日志来检查是否服务器确实消失了。  

       查看MySQL的正常运行时间（uptime）：show global status like 'uptime'; 

     查询错误日志的存储位置，然后打开错误日志，根据日志记录，确认是否存在服务器宕机。

     查询错误日志的存储位置：show variables like '%error';     

       如果MySQL服务器确实消失了，它是关闭了还是崩溃了，MySQL的错误日志会提供答案。通常MySQL的守护程序(mysqld)将由mysqld_safe包装器进程重新启动。

 2. 连接超时

     查看各项连接时间：show global variables like '%timeout';

      这些值是相对是MySQL的默认值，但是如果你的超时时间很短，则可能会出现这个错误，比如：

3. 你的SQL语句被杀死了

     有些系统会主动杀死运行时间过长的SQL语句，我们可以通过查看已经执行的kill语句数量来检查是否可能发生这种情况。

     查看mysql请求连接进程被主动杀死：show global  status like 'com_kill';

4. 你的SQL语句太大了

     稍微难以测试和验证，但是MySQL使用最大数据包站站点进行服务器和客户端之间的通信。如果语句包含大字段，则可能由于SQL语句的大小，而被中止。

      我们可以通过语句查看一下允许的最大包大小：show global variables like 'max_allowed_packet';   (1024*1024*5=5242880)

     如果值比较小，可以设置大一点：set global max_allowed_packet=1024*1024*16; 如果修改后不够大，可以继续加大。

​    

    注：通过命令行设置的大小仅对本次的有效，重启后就会回归原始值。通过修改配置文件（my.ini）则可以永久的设置参数。(C:\ProgramData\MySQL\MySQL Server)
