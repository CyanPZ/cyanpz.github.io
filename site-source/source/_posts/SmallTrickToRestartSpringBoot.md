---
title: 重启/关闭SpringBoot
date: 2018-09-01 23:20:24
tags:
---
# 重启/关闭SpringBoot的姿势

## 我们如何关闭SpringBoot程序?

- 首先找到程序的PID `ps aux | grep your-app.jar`

- 然后 `kill PID` or `kill -9 PID`

## 我们如何重启SpringBoot程序?

- 首先 关闭它

- 然后启动它(废话)

## 能不能优雅点？

- 首先在启动类加入ApplicationPidFileWriter

```java
SpringApplication app=new SpringApplication(DemoApplication.class);
app.addListeners(new ApplicationPidFileWriter("app.pid"));
app.run(args);
```

加入这个Listener在SpringBoot程序启动时，SpringBoot会将当前程序的PID写入app.pid
如：
```bash
cyan@cyan-HP-ZBook-15 ~/Workspace/Idea/Demo/target  ls
demo.jar
cyan@cyan-HP-ZBook-15 ~/Workspace/Idea/Demo/target  java -jar demo.jar >log.log & 
[1] 25189
cyan@cyan-HP-ZBook-15 ~/Workspace/Idea/Demo/target  ls
app.pid  demo.jar  log.log
cyan@cyan-HP-ZBook-15 ~/Workspace/Idea/Demo/target  cat app.pid 
25189                                                               cyan@cyan-HP-ZBook-15 ~/Workspace/Idea/Demo/target  vim app.pid 
cyan@cyan-HP-ZBook-15 ~/Workspace/Idea/Demo/target  ps aux | grep demo.jar
cyan     25189 34.4  4.3 6060504 345536 pts/2  SNl  23:56   0:11 java -jar demo.jar
cyan     25270  0.0  0.0  15968  1008 pts/2    S+   23:57   0:00 grep --color=auto --exclude-dir=.bzr --exclude-dir=CVS --exclude-dir=.git --exclude-dir=.hg --exclude-dir=.svn demo.jar
````

- 此时我们就可以通过`cat app.pid`获得SpringBoot程序的PID

进一步可以使用`kill -9 $(cat app.pid)`来关闭SpringBoot程序

```bash
cyan@cyan-HP-ZBook-15 ~/Workspace/Idea/Demo/target kill -9 $(cat app.pid)
cyan@cyan-HP-ZBook-15 ~/Workspace/Idea/Demo/target
[1]  + 25189 killed     java -jar demo.jar > log.log
```