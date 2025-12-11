# 更新系统包

首先，确保系统包是最新的：

sudo apt update
sudo apt upgrade -y

# 2. 安装依赖包

安装 Docker 所需的依赖包：

sudo apt install -y apt-transport-https ca-certificates curl software-properties-common

# 3. 添加 Docker 官方 GPG 密钥

添加 Docker 的官方 GPG 密钥以确保下载的软件包是安全的：

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# 4. 添加 Docker 仓库

将 Docker 的稳定版仓库添加到 APT 源列表中：

echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# 5. 更新包索引

更新 APT 包索引以包含 Docker 仓库：

sudo apt update

# 6. 安装 Docker

安装 Docker CE（社区版）、Docker CLI 和 Containerd：

sudo apt install -y docker-ce docker-ce-cli containerd.io

# 7. 启动并启用 Docker 服务

启动 Docker 服务并设置为开机自启：

sudo systemctl start docker
sudo systemctl enable docker

> 注意如果你使用Windows的WSL安装的Ubuntu系统的话，无法执行以上命令，会出现：
> 
> System has not been booted with systemd as init system (PID 1). Can't operate.
> 
> 因为WSL系统中使用的是经过修改的 nftables，而 Docker 安装程序使用 iptables 进行 NAT。  
> 可以使用以下命令将系统切换回使用传统的 iptables：
> 
> sudo update-alternatives --set iptables /usr/sbin/iptables-legacy
> sudo update-alternatives --set ip6tables /usr/sbin/ip6tables-legacy
> 
> 最后启动Docker，使用`service`而不是systemctl
> 
> sudo service docker start

# 8. 验证安装

通过运行 `hello-world` 镜像来验证 Docker 是否安装成功：

sudo docker run hello-world

如果看到`Hello from Docker!`，说明 Docker 已成功安装并运行。

> （可选）以非 root 用户身份运行 Docker  
> 默认情况下，Docker 需要 `sudo` 权限。如果你希望以非 root 用户身份运行 Docker，可以将用户添加到 `docker` 组：
> 
> sudo usermod -aG docker $USER
> 
> 然后，注销并重新登录以应用更改。

# 9. （可选）安装 Docker Compose

Docker Compose 是一个用于定义和运行多容器 Docker 应用程序的工具。你可以通过以下命令安装：

sudo apt install -y docker-compose

或者，你也可以从 Docker 官方 GitHub 仓库下载最新版本的 Docker Compose：

sudo curl -L "https://github.com/docker/compose/releases/download/$(curl -s https://api.github.com/repos/docker/compose/releases/latest | grep -Po '"tag_name": "\K.*\d')" /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose