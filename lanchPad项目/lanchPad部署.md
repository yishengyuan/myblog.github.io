## 前端部署

启动本地节点

```
npx hardhat node
```

安装make环境 并执行这个脚本

```
make ido
```

后端部署 


启动mysql

```
cd /c2n-be 
mvn clean install  -Dmaven.test.skip=true
```

## docker构建dockerfile


执行docker构建 
```
docker run -d --name portal-api -p 8080:8080 mycompany/portal-api:1.0.0-
```                                                                                                                  


## 代码解释 
```js

# 代表开发环境
SPRING_PROFILES_ACTIVE=dev
# 时区 
TZ=Asia/Shanghai

# 私钥
OWNER_PRIVATE_KEY=0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80

# mysql  host.docker.internal指的是本地docker地址 
#DB_HOST=host.docker.internal:3306
DB_HOST=brewery-mysql:3306
DB_NAME=brewery
DB_USERNAME=root
DB_PWD=123456

#hardhat
CHAIN_NERWORK_HOST=http://host.docker.internal:8545

```
## 安装maven
编辑配置
nano ~/.zshrc

配置maven
export M2_HOME=/usr/local/apache-maven-x.x.x
export PATH=$PATH:$M2_HOME/bin

配置生效
source ~/.zshrc

查看版本 
mvn --version  

## 依赖报错 
![[Pasted image 20251110162711.png]]

我看了一下 node的版本居然是v25.0 版本太高了 

修改为18.0  
nvm use 1.0

需要固定版本 
nvm alias default v18.0









