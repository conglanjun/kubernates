# 第二章 使用kubernates和docker

## 2.1 创建，运行及共享容器镜像

### 2.1.1 安装docker并运行hello world容器

访问`https://docs.docker.com/engine/install/` 进行安装。

```shell
sudo apt-get update
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
sudo apt-get install docker-ce docker-ce-cli containerd.io
// 免sudo docker
sudo groupadd docker
sudo gpasswd -a ${USER} docker
sudo service docker restart
newgrp - docker
```

```shell
docker run busybox echo "Hello world"
----------------------------------------
Unable to find image 'busybox:latest' locally
latest: Pulling from library/busybox
ea97eb0eb3ec: Pull complete 
Digest: sha256:bde48e1751173b709090c2539fdf12d6ba64e88ec7a4301591227ce925f3c678
Status: Downloaded newer image for busybox:latest
Hello world
```

### 2.1.2 创建一个简单的Node.js应用

创建app.js文件

```javascript
const http = require('http');
const os = require('os');

console.log('Kubia server starting...');

var handler = function(request, response) {
  console.log("Received request from " + request.connection.remoteAddress);
  response.writeHead(200);
  response.end("You've hit " + os.hostname() + "\n");
};
var www = http.createServer(handler);
www.listen(8080);
```

### 2.1.3 为镜像创建Dockerfile

创建Dockerfile文件，包含构建镜像执行的命令。要和app.js在同一个目录。

```dockerfile
FROM node:7
ADD app.js /app.js
ENTRYPOINT ["node", "app.js"]
```

### 2.1.4 构建容器镜像

```shell
docker build -t kubia .
```

#### 镜像是如何构建的

构建过程不是docker客户端进行的，是将整个目录文件上传到docker守护进程并在那里进行的。docker客户端和守护进程不要求在同一台机器上。如果你在一台非linux操作系统中使用docker，客户端就运行在你的宿主操作系统上，但是守护进程运行在一个虚拟机内。构建目录中的文件都被上传到了守护进程中。

#### 镜像分层

镜像由很多层组成。每一层有一行pull complete，不同镜像会共享分层，会让存储和传输变得高效。dockerfile中每一条单独的指令到会创建一个新层。拉取基础镜像所有分层之后，docker在他们上面创建一个新层并且添加app.js。在创建另一层来指定镜像被运行时所执行的命令。最后一层被标记为`kubia:latest`。

```shell
docker images
--------------------
REPOSITORY   TAG       IMAGE ID       CREATED         SIZE
kubia        latest    07bd2bcd0ec6   7 minutes ago   660MB
busybox      latest    219ee5171f80   3 weeks ago     1.23MB
node         7         d9aed20b68a4   3 years ago     660MB
```

### 2.1.5 运行容器镜像

