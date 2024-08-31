# Docker

## docker安装
```shell
# 1、卸载旧版本 Docker
yum remove docker \
    docker-client \
    docker-client-latest \
    docker-common \
    docker-latest \
    docker-latest-logrotate \
    docker-logrotate \
    docker-engine

# 2、安装必要的一些系统工具
yum install -y yum-utils device-mapper-persistent-data lvm2

# 3、添加软件源信息，国内 Repository 更加稳定
yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

# 4、更新
	#centOS7:
	yum makecache fast
	#centOS8
	dnf makecache
	
# 5、安装最新版本的 Docker Engine 和 Container
yum install docker-ce docker-ce-cli containerd.io
	#注：安装指定版本的 Docker Engine
	## ①查找 docker-ce 的版本列表
	yum list docker-ce --showduplicates | sort -r
	## ②安装命令：显示结果：中间一列内容冒号和连字符之间的是版本号，例如，19.03.13-3 就是版本号  
	# yum install docker-ce-<VERSION_STRING> docker-ce-cli-<VERSION_STRING> containerd.io
	# 示例：
	yum install docker-ce-19.03.13-3.el8 docker-ce-cli-19.03.13-3.el8 containerd.io

# 6、配置国内镜像加速
	## ①创建目录
	mkdir -p /etc/docker
	## ②编写配置文件
	tee /etc/docker/daemon.json <<-'EOF'
	{
      "registry-mirrors": ["http://hub-mirror.c.163.com",
        "https://docker.mirrors.ustc.edu.cn",
        "https://reg-mirror.qiniu.com",
        "http://f1361db2.m.daocloud.io"
  		]
	}
	EOF

# 7、开机自启
systemctl enable docker
# 启动
systemctl start docker
```
## docker命令
```shell
#查看docker状态
systemctl status docker
#关闭docker
systemctl stop docker

##docker重启
systemctl restart docker

##查看已安装的镜像
docker images

##docker中 搜索镜像
docker search 镜像名称

#拉取一个镜像到本地
docker pull 镜像名称

##查看已运行的容器实例
docker ps

##删除指定镜像
docker rmi image_name

##docker中 删除所有的镜像
docker rmi $(docker images | awk '{print $3}' |tail -n +2)

#运行容器
## -d 后台运行
## --name java8 容器名，自定义的
## -p 8080:80   端口进行映射，将本地 8080 端口映射到容器内部的 80 端口
docker run -d --name nginx -p 8080:80 nginx

##暂停容器
docker pause 容器id / 容器名
##终止暂停容器
docker unpause 容器id / 容器名

#启动容器实例
docker start 容器id / 容器名

##docker中 启动所有的容器命令
docker start $(docker ps -a | awk '{ print $1}' | tail -n +2)

#关闭容器实例
docker stop  容器id / 容器名

##docker中 关闭所有的容器命令
docker stop $(docker ps -a | awk '{ print $1}' | tail -n +2)

##docker中 删除所有的容器命令
docker rm $(docker ps -a | awk '{ print $1}' | tail -n +2)

##移除容器
docker rm 容器id / 容器名


```