wsl中使用的docker就是和windows一致的 

docker安装mysql
~~~
docker pull mysql:8.0
~~~


执行版本执行

```
docker run -t -i mysql:8.0 /bin/bash
```

```
查看容器
docker ps -a  
```

```
进入容器   50c4c9700775  为容器id
docker exec -it 50c4c9700775 /bin/bash
```

要退出终端，直接输入 **exit**:

```
root@ed09e4490c57:/# exit
```

查看镜像

```
docker images
```

查看端口映射

```
docker port 50c4c9700775 
```

## 安装mysql

```
docker run -itd --name mysql-test -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 mysql
```

进入mysql容器

```
mysql -h localhost -u root -p
```



