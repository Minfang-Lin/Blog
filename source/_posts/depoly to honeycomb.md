---
title: 部署JavaWeb项目到蜂巢时的注意事项
date: 2016-11-19 14:00
tags: JavaWeb
---

## 数据库相关配置

蜂巢默认没有初始密码，先通过mysql命令进入数据库管理，执行以下语句将root密码设置为123456。
``` sql
select user();
set password=password('123456'); 
flush privileges; 
```

让root帐号获取所有访问权限。
``` sql
grant all on *.* to root@'127.0.0.1'identified by "123456";
```

<!--more-->

## 更换maven镜像为阿里云

寻找maven的settings.xml目录
``` bash
find / -name "settings.xml"
```

使用vim编辑
``` bash
vim /etc/maven/settings.xml
```

在`<mirrors>`元素里面加一个`<mirror>`配置
``` xml
<mirror>
    <id>aliyun</id>
    <mirrorOf>centeral</mirrorOf>
    <name>aliyun mirror</name>
    <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
</mirror>
```
在`<profiles>`中加一个`<profile>`配置
``` xml
<profile>
    <id>aliyun</id>
    <activation>
        <activeByDefault>true</activeByDefault>
    </activation>

    <repositories>
        <repository>
            <id>aliyun</id>
	    <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
        </repository>
    </repositories>

    <pluginRepositories>
            <pluginRepository>
                <id>aliyun</id>
	        <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
            </pluginRepository>
      </pluginRepositories>
</profile>
```

## 解决OOM

默认的容器分配给tomcat的内存不足，如果部署完成以后页面一直转呀转不出来，很有可能是这个问题。可以看下tomcat的日志确定一下。

``` bash
tail /var/log/tomcat7/catalina.out
```

如果看到类似下面的报错，就说明是OOM了。

``` bash
SEVERE: Error waiting for multi-thread deployment of WAR files to complete                                    
java.util.concurrent.ExecutionException: java.lang.OutOfMemoryError: Java heap space                          
        at java.util.concurrent.FutureTask$Sync.innerGet(FutureTask.java:252)                                 
        at java.util.concurrent.FutureTask.get(FutureTask.java:111)                                           
        at org.apache.catalina.startup.HostConfig.deployWARs(HostConfig.java:752)                             
        at org.apache.catalina.startup.HostConfig.deployApps(HostConfig.java:472)                             
        at org.apache.catalina.startup.HostConfig.check(HostConfig.java:1454)                                 
        at org.apache.catalina.startup.HostConfig.lifecycleEvent(HostConfig.java:296)                         
        at org.apache.catalina.util.LifecycleSupport.fireLifecycleEvent(LifecycleSupport.java:119)            
        at org.apache.catalina.util.LifecycleBase.fireLifecycleEvent(LifecycleBase.java:90)
```

修改方法：打开文件`/usr/share/tomcat7/bin/catalina.sh`

``` bash
# OS specific support.  $var _must_ be set to either true or false.
--这个位置--
cygwin=false
darwin=false
```

 在“这个位置”添加`JAVA_OPTS="-Xms256m -Xmx512m -Xss1024K -XX:PermSize=128m -XX:MaxPermSize=256m"`，即

``` bash
# OS specific support.  $var _must_ be set to either true or false. 
JAVA_OPTS="-Xms256m -Xmx512m -Xss1024K -XX:PermSize=128m -XX:MaxPermSize=256m"
cygwin=false
darwin=false
```

保存文件后重启tomcat服务。