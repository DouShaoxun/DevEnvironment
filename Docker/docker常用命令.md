





```
1. --name:为容器指定名称
2. -it:启动一个交互型容器，此参数为我们和容器提供了一个交互shell
3. -d:创建后台型容器
4. -restart=always:容器退出后自动重启
5. -restart=on-failure:x:容器退出时如果返回值是非0，就会尝试重启x次
6. -p x:y :主机端口：容器端口
7. -P：随机分配一个49000到49900的端口
8.-v：创建数据卷
7. -n :指定dns
8. -h : 指定容器的hostname
9. -e ：设置环境变量
10. -m :设置容器使用内存最大值
11. --net: 指定容器的网络连接类型，支持 bridge/host/none/container
12. --link=x: 添加链接到另一个容器x
13. --expose=x: 开放端口x
```



```shell
查询容器独立ip
docker inspect --format='{{.NetworkSettings.IPAddress}}' 容器id或者名称
```

```shell
# 从容器中拷贝文件到宿主机
docker cp mysql801:/etc/mysql/my.cnf /opt/dockerSoftware/my.cnf

# 从宿主机拷贝文件到容器
docker cp /opt/dockerSoftware/my.cnf mysql802:/etc/mysql/my.cnf
```

