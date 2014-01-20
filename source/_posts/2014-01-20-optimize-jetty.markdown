---
layout: post
title: "优化Jetty"
date: 2014-01-20 13:41:03 +0800
comments: true
categories: Jetty
---


##调整Linux内核参数

#####调整Tcp Buffer Sizes
```
net.core.rmem_max = 16777216
net.core.wmem_max = 16777216
net.ipv4.tcp_rmem = 4096 87380 4194304
net.ipv4.tcp_wmem = 4096 16384 4194304
```

####调整Queue Sizes

```
net.core.somaxconn = 4096

net.core.netdev_max_backlog = 16385
net.ipv4.tcp_max_syn_backlog = 8192
net.ipv4.tcp_syncookies = 1
```

####调整Ports

```
net.ipv4.ip_local_port_range = 1024 65000
net.ipv4.tcp_tw_recycle = 1
```
---
##调整JVM参数
---


##调整Jetty配置

####调整Acceptors数量
大于等于1，小于等于CPU数量

```
<Set name="Acceptors">4</Set>
```

####为线程池的阻塞队列设置默认大小
默认大小的值以每分钟的请求数来计算

```
<Set name="ThreadPool">
      <!-- Default queued blocking threadpool -->
      <New class="org.eclipse.jetty.util.thread.QueuedThreadPool">
        <Arg>
          <New class="java.util.concurrent.ArrayBlockingQueue">
            <Arg type="int">6000</Arg>
          </New>
        </Arg>
        <Set name="minThreads">10</Set>
        <Set name="maxThreads">500</Set>
        <Set name="detailedDump">false</Set>
      </New>
    </Set>
```