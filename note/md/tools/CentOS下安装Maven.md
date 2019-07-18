1. 确保安装了jdk
<br/>
2. 下载maven
``` bash
[root@localhost ~]# https://mirrors.tuna.tsinghua.edu.cn/apache/maven/maven-3/3.6.0/binaries/apache-maven-3.6.0-bin.tar.gz
```
3. 解压Mavne文件
``` bash
# 新建文件夹
[root@localhost ~]# mkdir /usr/local/maven
# 解压到指定目录
[root@localhost ~]# tar -zxvf apache-maven-3.6.0-bin.tar.gz -C /usr/local/maven/
```
4. 配置Maven环境变量
``` bash
[root@localhost ~]# vi /etc/profile
export MAVEN_HOME=/usr/local/maven/apache-maven-3.6.0
export PATH=$PATH:$MAVEN_HOME/bin
```
5. 使文件生效
``` bash
[root@localhost ~]# source /etc/profile
```
6. 测试
``` bash
[root@localhost ~]# mvn -version
Apache Maven 3.6.0 (97c98ec64a1fdfee7767ce5ffb20918da4f719f3; 2018-10-25T02:41:47+08:00)
Maven home: /usr/local/maven/apache-maven-3.6.0
Java version: 1.8.0_191, vendor: Oracle Corporation, runtime: /usr/local/java/jdk1.8.0_191/jre
Default locale: en_US, platform encoding: UTF-8
OS name: "linux", version: "3.10.0-862.el7.x86_64", arch: "amd64", family: "unix"
```
