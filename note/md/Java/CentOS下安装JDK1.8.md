1. [下载jdk1.8](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html  )压缩文件  
<br/>
2. 解压
``` bash
# 创建文件
mkdir /usr/local/java
# 解压
tar -zxvf filename -C /usr/local/java
```
3. 编辑/etc/下的profile文件，配置环境变量
``` bash
[root@bogon software]# vim /etc/profile
# jdk1.8.0_101根据具体的小版本修改
export JAVA_HOME=/usr/local/java/jdk1.8.0_101
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export PATH=${JAVA_HOME}/bin:$PATH
```
4. 使/etc/profile生效
``` bash
[root@bogon jdk1.8.0_101]# source /etc/profile
```
5. 测试
``` bash
[root@bogon jdk1.8.0_101]# java -version
java version "1.8.0_101"
Java(TM) SE Runtime Environment (build 1.8.0_101-b13)
Java HotSpot(TM) 64-Bit Server VM (build 25.101-b13, mixed mode)
```