## CentOS 安装Docker

1、yum 包更新到最新 

```
yum update
```

2、安装需要的软件包， yum-util 提供yum-config-manager功能，另外两个是devicemapper驱动依赖的

```
yum install -y yum-utils device-mapper-persistent-data lvm2
```

3、 设置yum源

```
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

4、 安装docker

```
yum install -y docker-ce
```

5.查看docker版本，验证是否验证成功

```
docker -v
```

6.设置自启动等

```shell
# 启动docker服务
systemctl start docker
# 设置开机自启docker
systemctl enable docker
# 重启docker命令
systemctl restart docker
# 停止docker服务
systemctl stop docker
#安装完成后打开阿里云设置镜像加速
```