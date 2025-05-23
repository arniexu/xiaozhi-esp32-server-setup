# xiaozhi-esp32-server-setup

## 维护人

xuqianjin123@126.com

call me +86-13262637680

## 前提

有一台自己的linux机器，无论是虚拟机还是云服务器都可以，我这是基于虚拟机做的。

## 第0步，搭建自己的虚拟机或者云服务器

这一步不是我们本次讨论的内容。

## 第一步，下载安装docker

这一部没有什么值得细说的内容，这一步非常简单。

```bash
 curl -fsSL https://test.docker.com -o test-docker.sh
 sudo sh test-docker.sh
# 通过下面命令验证这一步是否成功
# sudo docker -v
# 如果正常输出版本信息那就是成功了

```

## 第二步，安装redis和数据库

### 如果docker镜像拉取失败，一般是镜像源的问题，可以将镜像源换成国内的镜像源。

```bash
# 修改/etc/docker/daemon.json
vim /etc/docker/daemon.json
# 重啓docker服務
systemctl daemon-reload
systemctl restart docker
```

### 那么我该怎么判断我现在的失败是由于镜像源访问失败导致的呢？可以用下面的命令測試。

```bash
docker pull hello-world
```

### 鏡像源我究竟有那些選擇呢？

1.Docker中国区官方镜像   [https://registry.docker-cn.com](https://registry.docker-cn.com)

2.网易  [http://hub-mirror.c.163.com](http://hub-mirror.c.163.com)

3.ustc [https://docker.mirrors.ustc.edu.cn](https://docker.mirrors.ustc.edu.cn)

4.中国科技大学 [https://docker.mirrors.ustc.edu.cn](https://docker.mirrors.ustc.edu.cn)

5.阿里云容器 生成自己的加速地址

登录：cr.console.aliyun.com，点击“创建我的容器镜像”，得到专属加速地址。

### 部署小智数据库和redis

```bash
sudo docker run --name xiaozhi-esp32-server-db -e MYSQL_ROOT_PASSWORD=123456 -p 3306:3306 -e MYSQL_DATABASE=xiaozhi_esp32_server -e MYSQL_INITDB_ARGS="--character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci" -d mysql:latest
sudo docker run --name xiaozhi-esp32-server-redis -d -p 6379:6379 redis
# 通过下面命令验证这一步是否成功
# sudo docker ps
# 如果输出是这样类似的的，那就是成功了
# CONTAINER ID   IMAGE          COMMAND                  CREATED       STATUS       PORTS                                                    NAMES
# 94d02c8bb0da   redis          "docker-entrypoint.s…"   6 hours ago   Up 6 hours   0.0.0.0:6379->6379/tcp, [::]:6379-#>6379/tcp              xiaozhi-esp32-server-redis
# c9285d50975f   mysql:latest   "docker-entrypoint.s…"   6 hours ago   Up 6 hours   0.0.0.0:3306->3306/tcp, [::]:3306->3306/tcp, 33060/tcp   xiaozhi-esp32-server-db
```

## 第三步，安装manager-api

这一步也很简单的基本不会出什么错误，直接运行下面的脚本就可以将所需依赖安装完成。

```bash
#!/bin/bash

# 更新系统软件包列表
sudo apt update
if [ $? -ne 0 ]; then
    echo "更新软件包列表失败，请检查网络或权限。"
    exit 1
fi

# 安装JDK 21
sudo apt install -y openjdk-21-jdk
if [ $? -ne 0 ]; then
    echo "安装JDK 21失败，请检查网络或权限。"
    exit 1
fi

# 设置JDK环境变量
echo 'export JAVA_HOME=/usr/lib/jvm/java-21-openjdk-amd64' >> ~/.bashrc
echo 'export PATH=$PATH:$JAVA_HOME/bin' >> ~/.bashrc

# 安装Maven
sudo apt install -y maven
if [ $? -ne 0 ]; then
    echo "安装Maven失败，请检查网络或权限。"
    exit 1
fi

# 设置Maven环境变量
echo 'export MAVEN_HOME=/usr/share/maven' >> ~/.bashrc
echo 'export PATH=$PATH:$MAVEN_HOME/bin' >> ~/.bashrc

# 添加VSCode仓库并安装VSCode
wget -qO- https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > packages.microsoft.gpg
sudo install -D -o root -g root -m 644 packages.microsoft.gpg /etc/apt/keyrings/packages.microsoft.gpg
sudo sh -c 'echo "deb [arch=amd64,arm64,armhf signed-by=/etc/apt/keyrings/packages.microsoft.gpg] https://packages.microsoft.com/repos/code stable main" > /etc/apt/sources.list.d/vscode.list'
rm -f packages.microsoft.gpg

sudo apt update
if [ $? -ne 0 ]; then
    echo "更新VSCode软件包列表失败，请检查网络或权限。"
    exit 1
fi

sudo apt install -y code
if [ $? -ne 0 ]; then
    echo "安装VSCode失败，请检查网络或权限。"
    exit 1
fi

# 使环境变量生效
source ~/.bashrc

echo "JDK 21、Maven和VSCode已成功安装，环境变量已设置。"  

# 通过下面的命令检查java环境是否安装成功
# javac -v
# 查看ubuntu下面是否已经装好vscode，并且使用vscode打开manager-api可以正常运行（run->run without debugging）
```

### 启动manager-api

做这一步的前提是先从网上拉取xiaozhi-esp32-server的源代码，通过下面的命令拉取

```bash
git clone https://github.com/xinnan-tech/xiaozhi-esp32-server.git 
```

拉取完成后，进入xiaozhi-esp32-server/main/manager-api目录下执行

```bash
sudo mvn spring-boot:run
# 如果出现下属日志输出，则manager-api启动正常
# 2025-xx-xx 22:11:12.445 [main] INFO  c.a.d.s.b.a.DruidDataSourceAutoConfigure - Init DruidDataSource
# 2025-xx-xx 21:28:53.873 [main] INFO  xiaozhi.AdminApplication - Started AdminApplication in 16.057 seconds (process running for 17.941)
# http://localhost:8002/xiaozhi/doc.html

```

## 第五步，安装manager-web

还是老套路，先安装依赖

```bash
#!/bin/bash

# 更新系统软件包列表
sudo apt update
if [ $? -ne 0 ]; then
    echo "更新软件包列表失败，请检查网络或权限。"
    exit 1
fi

# 安装必要的依赖
sudo apt install -y curl
if [ $? -ne 0 ]; then
    echo "安装curl失败，请检查网络或权限。"
    exit 1
fi

# 使用NodeSource仓库安装Node.js
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
if [ $? -ne 0 ]; then
    echo "添加NodeSource仓库失败，请检查网络或权限。"
    exit 1
fi

# 安装Node.js和npm
sudo apt install -y nodejs
if [ $? -ne 0 ]; then
    echo "安装Node.js和npm失败，请检查网络或权限。"
    exit 1
fi

# 验证安装结果
node --version
npm --version

echo "Node.js和npm已成功安装。"
```

### 启动manager-web

进入xiaozhi-esp32-server/main/manager-web文件夹，执行

```bash
sudo npm install
sudo npm run serve
```

## 第七步，尝试登录小智管理页面

当manager-web启动后，小智的网页服务器已经在正常工作了，我们现在已经可以正常的登录小智管理网页了。

![1745882037267](images/README/1745882037267.png)

### 7.1 注册超级用户

小智注册的第一个用户将会是超级用户。

### 7.2 配置secret key

登录上网页后，需要将网页上的secret.key拷贝到xiaozhi-server/data/.config.yaml的秘钥字段

## 第八步，运行xiaozhi-server

### 下载声音模型

下载声音模型到xiaozhi-esp32-server/main/xiaozhi-server/models/SenseVoiceSmall

```bash
wget https://modelscope.cn/models/iic/SenseVoiceSmall/resolve/master/model.pt
```

### 下载配置文件

下载配置文件到xiaozhi-esp32-server/main/xiaozhi-server/data

```bash
wget https://github.com/xinnan-tech/xiaozhi-esp32-server/blob/14a63c4b97a0640dc322e50d181f45eabb7d3ddd/main/xiaozhi-server/config_from_api.yaml
mv config_from_api.yaml .config.yaml
```

### 配置secret-key

将http://127.0.0.1:8001参数管理server.secret_key 拷贝到上面的配置文件之中去

配置文件长这个样子

```text
manager-api:
  url: http://127.0.0.1:8002/xiaozhi
  secret: 12345678-xxxx-xxxx-xxxx-123456789000
```

### 配置大模型的API秘钥

根據http://127.0.0.1:8001中配置的大模型，你需要去他的官網上申請API，這個過程是要付費的。

### 安装python运行环境

```bash

#!/bin/bash

# 更新系统软件包列表
sudo apt update
if [ $? -ne 0 ]; then
    echo "更新软件包列表失败，请检查网络或权限。"
    exit 1
fi

# 安装 wget
sudo apt install -y wget
if [ $? -ne 0 ]; then
    echo "安装 wget 失败，请检查网络或权限。"
    exit 1
fi

# 下载并安装 Miniconda
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
if [ $? -ne 0 ]; then
    echo "下载 Miniconda 安装脚本失败，请检查网络。"
    exit 1
fi

bash Miniconda3-latest-Linux-x86_64.sh -b -p $HOME/miniconda
if [ $? -ne 0 ]; then
    echo "安装 Miniconda 失败，请检查权限。"
    exit 1
fi

# 初始化 Conda
$HOME/miniconda/bin/conda init bash
source ~/.bashrc

# 安装 libopus
sudo apt install -y libopus0
if [ $? -ne 0 ]; then
    echo "安装 libopus 失败，请检查网络或权限。"
    exit 1
fi

# 安装 ffmpeg
sudo apt install -y ffmpeg
if [ $? -ne 0 ]; then
    echo "安装 ffmpeg 失败，请检查网络或权限。"
    exit 1
fi

# 清理下载的 Miniconda 安装脚本
rm Miniconda3-latest-Linux-x86_64.sh

echo "Conda、libopus 和 ffmpeg 已成功安装。"


conda remove -n xiaozhi-esp32-server --all -y
conda create -n xiaozhi-esp32-server python=3.10 -y
conda activate xiaozhi-esp32-server

# 添加清华源通道
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge

conda install libopus -y
conda install ffmpeg -y
```

## 第十步，进入python虚拟环境，启动 esp32-server

```bash
#!/bin/sh
#
conda remove -n xiaozhi-esp32-server --all -y
conda create -n xiaozhi-esp32-server python=3.10 -y
conda activate xiaozhi-esp32-server

# 添加清华源通道
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge

conda install libopus -y
conda install ffmpeg -y

conda activate xiaozhi-esp32-server
cd /home/xuqianjin/xiaozhi-esp32-server/main/xiaozhi-server
pip config set global.index-url https://mirrors.aliyun.com/pypi/simple/
pip install -r requirements.txt

echo download the voice model and put it into the correct path
wget https://modelscope.cn/models/iic/SenseVoiceSmall/resolve/master/model.pt
mmv model.pt 
mv model.pt models/SenseVoiceSmall/


echo download the config file and replace the secret key 
https://github.com/xinnan-tech/xiaozhi-esp32-server/blob/main/main/xiaozhi-server/config_from_api.yaml
mv config_from_api.yaml ./data/.config.yaml


```

## 第十一步·，检查小智联网方式，编译小智，烧录

## 第十二步，绑定设备到后台网页
