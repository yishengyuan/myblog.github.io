docker-compose up -d   
启动 Docker Compose 配置中的所有服务的命令。 


- **读取 `docker-compose.yml` 文件**：它会查找当前目录中的 `docker-compose.yml` 文件，这个文件定义了服务、网络、卷和其他配置。
    
- **拉取镜像**：如果配置文件中定义的服务所需的镜像本地不存在，Docker Compose 会尝试从 Docker Hub 或其他镜像仓库拉取相应的镜像。
    
- **创建和启动容器**：根据配置文件中的定义，Docker Compose 会创建并启动一个或多个容器。如果有依赖关系（例如，数据库和应用服务），它会按顺序启动。
    
- **创建网络**：如果在配置文件中定义了网络，Docker Compose 会创建这些网络，确保服务可以在这些网络中相互通信。
    
- **创建卷**：如果配置中指定了卷（用于持久化数据），Docker Compose 会创建这些卷。
    
- **后台运行容器**：使用 `-d` 参数（即“detached”模式），容器会在后台运行，而不会占用当前终端窗口。这意味着你不会看到容器的实时输出，除非你使用 `docker logs` 查看。

如果发现内存不够用的情况下。或者是macos遇到jdk兼容问题 这就是一个方案 
**在本地 IntelliJ IDEA 中配置 Docker 容器中的 JDK** 是最简便、最常见的方案 

## docker构建 


通过dockercompose构建  可以将每个服务的关系 关联起来进行管理 

![[Pasted image 20251110124914.png]]

## docker-compose.yml  

```
mysql-db:  
  container_name: bizzan-mysql  
  image: library/mysql:5.7.29  
  platform: linux/amd64  
  restart: always  
  volumes:  
    - ./sql/my.conf:/etc/my.conf  
    - ./sql/init:/docker-entrypoint-initdb.d/  
    - ./sql/data:/var/lib/mysql  
  ports:  
    - "3306:3306"  
  networks:  
    - app-network  
  environment:  
    MYSQL_ROOT_PASSWORD: 123456  
    MYSQL_DATABASE: bizzan  
    MYSQL_LOWER_CASE_TABLE_NAMES: 1
```

volumes其实就是挂载的意思 
- ./sql/my.conf:/etc/my.conf   配置文件会使用本地文件的/sql/my.conf  
- ./sql/init:/docker-entrypoint-initdb.d/    初始化脚本会使用./sql/init: 文件夹下的 
- ./sql/data:/var/lib/mysql   `./sql/data:/var/lib/mysql` 
- 这个挂载的目的是将宿主机的 `./sql/data` 目录挂载到容器内的 `/var/lib/mysql` 目录，这样做是为了 **持久化 MySQL 的数据**。 

## Dockerfile.cloud 
```js
#从构建阶段复制JAR文件 
COPY --from=builder /build/cloud/target/*.jar /app.jar 
ENV TZ=Asia/Shanghai 
EXPOSE 7421 
CMD ["java", "-Djava.security.egd=file:/dev/./urandom", "-jar", "/app.jar"]
```


COPY --from=builder /build/cloud/target/*.jar /app.jar   
拷贝构建下的jar文件 并重命名为app.jar

ENV TZ=Asia/Shanghai   
时区设置为亚洲上海

EXPOSE 7421   
暴露端口7421 

CMD ["java", "-Djava.security.egd=file:/dev/./urandom", "-jar", "/app.jar"]  
执行 java命令 java -jar 

## zookeeper 使用了bitnami 为什么要使用这个版本 ？ 
![[Pasted image 20251110130609.png]]
但是无法使用的状态


![[5d35bfc6c561c09a67b2bbe7008c9d89.png]]

先部署成功 然后再看后台的撮合机制


## 镜像不可用 
![[Pasted image 20251110173332.png]] 
![[ead59c2b795fbbc92655b2803bf64ee4.png]]

需要查一下是否可以再使用 因为不再维护了。所以无法使用了
可以切换到其他版本


## docker版本查看 
![[Pasted image 20251110175128.png]]

11-ea-17-jdk-slim

![[Pasted image 20251110175221.png]]


![[Pasted image 20251110180338.png]]

![[Pasted image 20251110180403.png]]




![[Pasted image 20251110185642.png]]

zookeeper
docker pull swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/bitnami/zookeeper:3.8 docker tag swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/bitnami/zookeeper:3.8 docker.io/bitnami/zookeeper:3.8




![[Pasted image 20251111150206.png]]

|命令|作用|
|---|---|
|`docker compose down`|停止并删除容器、网络（默认不删除数据卷和镜像）|
|`docker compose stop`|仅停止容器，不删除|
|`docker compose down -v`|停止并删除容器、网络和数据卷|
|`docker compose down --rmi all`|同时删除相关镜像|
	