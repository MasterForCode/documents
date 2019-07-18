> 查看CentOS版本
``` bash
# 3.10以下版本不支持
[root@localhost ~]# uname -r
3.10.0-862.el7.x86_64
```

> 更新系统（建议）
``` bash
[root@localhost ~]# yum update
```
> 删除旧版本
``` bash
[root@localhost ~]# yum remove docker \
>                   docker-client \
>                   docker-client-latest \
>                   docker-common \
>                   docker-latest \
>                   docker-latest-logrotate \
>                   docker-logrotate \
>                   docker-selinux \
>                   docker-engine-selinux \
>                   docker-engine
```

> 安装必要包
``` bash
# yum-utils提供yum-config-manager实用程序，devicemapper存储驱动程序需要device-mapper-persistent-data和lvm2。
[root@localhost ~]# yum install -y yum-utils \
>   device-mapper-persistent-data \
>   lvm2
```

> 设置仓库
``` bash
[root@localhost ~]# yum-config-manager \
>     --add-repo \
>     https://download.docker.com/linux/centos/docker-ce.repo
```

> 可选：启用边缘和测试存储库。这些存储库包含在上面的docker.repo文件中，但默认情况下处于禁用状态。您可以将它们与稳定存储库一起启用。
``` bash
[root@localhost ~]# yum-config-manager --enable docker-ce-edge
[root@localhost ~]# yum-config-manager --enable docker-ce-test
# 禁用示例
[root@localhost ~]# yum-config-manager --disable docker-ce-edge
```

> 安装docker(最新版本)
``` bash
[root@localhost ~]# yum install docker-ce
```

> 启动docker
``` bash
[root@localhost ~]# systemctl start docker
```

> 验证docker是否安装完成
``` bash
[root@localhost ~]# docker run hello-world
```

> 为了避免（普通用户）每次使用docker命令都使用sudo，添加docker用户和用户组
``` bash
# 添加docker用户
[root@localhost ~]# useradd -g docker docker
# 添加docker用户组，your-name为赋予操作权限的用户
[root@localhost ~]# usermod -aG docker your-user
```

> 开机自启
``` bash
[root@localhost ~]# systemctl enable docker.service
```
> 卸载

* 查看安装的软件包

``` bash
[root@localhost ~]# yum list installed | grep docker
```
* 删除安装的软件
``` bash
# 示例
[root@localhost ~]# yum -y remove docker-engine.x86_64
# and so on
```
* 删除所有镜像，容器和卷组
``` bash
# 查看docker相关文件及文件夹
[root@localhost ~]# find | grep docker
# 示例
[root@localhost ~]# rm -rf /var/lib/docker
# and so on
```
* 删除用户用户组配置信息