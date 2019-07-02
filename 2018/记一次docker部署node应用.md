### 前言

由于服务器到期，过往部署好的项目需要搬迁。搬迁过程中，深感配环境装依赖等枯燥无聊，要是能项目+环境一块搬迁就好，因此想到使用`docker`，满足一切愿望

### 理解 docker

个人理解类似虚拟机，能将各种软件等一块复制去别的电脑运行。但又没有虚拟机占用资源多、启动慢、体积庞大等问题。且`docker`运行在虚拟容器中，同一台服务器的项目环境不会相互冲突

### 安装

不同系统安装方法有所不同， 我的服务器为`centOS`，安装方法如下

```bash
sudo yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2

sudo yum-config-manager \
  --add-repo \
  https://download.docker.com/linux/centos/docker-ce.repo

sudo yum install docker-ce
```

### 部署`node`进程

- 启动`docker`, `sudo systemctl start docker`
- 下载`nodejs`某版本的`docker image`，例如我需要 8.7，`docker images pull node:8.7.0`
- 运行第二步的下载好的`node images`，并将端口映射到`node`进程监听的端口上，
  例如`docker container run -p 5050:5050 -it node:8.7 /bin/bash`，`-p`表示将`images`内部 5050 端口(右)映射到系统 5050 端口(左)，`/bin/bash`表示自动进入`shell`命令
- 下载你的`node`项目，例如`git clone ***`，并安装相关依赖
- 使用`pm2`将`node`进程变为后台服务，使得`images`退出后项目依旧运行
  - `npm install pm2 -g`
  - `pm2 start xx.js -i max`

### 重启`docker`中的`node`进程

- `docker container ls`，找到输出信息中的`NAMES`，例如`zealous_wozniak`
- `docker exec -it zealous_wozniak /bin/bash`，根据`NAMES`进入对应`docker images`
- `pm2 list`，找到输出信息中的`App name`，例如`app`
- `pm2 restart app`，重启

### container 相关

```
docker start [NAMES]
docker stop [NAMES]
docker restart [NAMES]
```

### 其他

也可以使用`Dockerfile`创建项目自身的`docker iamges`，但个人觉得维护起来很麻烦，每次更新代码都需要重新`build images`，浪费不少时间。目前很多第三方`docker images`，一般情况下直接`pull`相应的就好

### 相关资料

- [阮一峰 docker 入门](http://www.ruanyifeng.com/blog/2018/02/docker-tutorial.html)
- [centOS 安装 docker 官方教程](https://docs.docker.com/install/linux/docker-ce/centos/)
