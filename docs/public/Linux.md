# Linux

## 查看Linux系统版本信息

### 查看Linux内核版本
> cat /proc/version  

> uname -a

### 查看Linux系统版本的命令
```shell
lsb_release -a   #适用于所有的Linux发行版,需要安装命令： yum install -y redhat-lsb-core
```


## 修改端口号
1. 修改sshd_config
> vim /etc/ssh/sshd_config
```shell
Port 22 #默认使用22端口，也可以自行修改为其他端口，但登录时要打上端口号
#ListenAddress #指定提供ssh服务的IP，这里我注释掉。
PermitRootLogin #禁止以root远程登录
PasswordAuthentication yes #启用口令验证方式
PermitEmptyPassword #禁止使用空密码登录
LoginGraceTime 1m #重复验证时间为1分钟
MaxAuthTimes 3 #最大重试验证次数
```
2. 重启sshd服务
> systemctl restart sshd.service

## Linux清理空间
> 1.df -h ，这个命令用于查看服务器空间  
> 2.du -h --max-depth=1，这个命令用于查看当前目录，哪个文件占用最大  
> 或 du -sh *，这个命令也用于查看当前目录下各文件及文件夹占用大小  
> 3.确定某文件，执行删除或置空文件

## 查看端口
> lsof -i:端口号

> netstat -tunlp|grep 端口号

## Linux置空文件命令
> `> console.log`

> `cp /dev/null console.log`

> `cat /dev/null > anaconda-ks.cfg`


## 防火墙
```shell
firewall-cmd --list-ports  #查看已开放的端口
#开放端口（开放后需要要重启防火墙才生效）  
firewall-cmd --zone=public --add-port=18919/tcp --permanent
#关闭端口（关闭后需要要重启防火墙才生效）  
firewall-cmd --zone=public --remove-port=6379/tcp --permanent
systemctl enable firewalld #开机启动防火墙
systemctl disable firewalld #禁止防火墙开机启动 
systemctl start firewalld #开启防火墙
systemctl stop firewalld #停止防火墙
systemctl restart firewalld #重启防火墙
```

## linux 配置jdk
1. 下载[jdk](https://www.oracle.com/java/technologies/javase/javase8-archive-downloads.html),解压  
> tar -zxvf jdk-8u181-linux-x64.tar.gz
2. 配置环境变量  
> vim /etc/profile  
```shell 
#jdk
export JAVA_HOME=/usr/local/java/jdk1.8.0_271
export PATH=$PATH:$JAVA_HOME/bin
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
```
3. 刷新配置  
> source /etc/profile

## 切换Linux镜像源
1. 备份原配置，下载新配置,[阿里镜像](https://developer.aliyun.com/mirror/?spm=a2c6h.13651104.0.d1002.6dbd12b2X0pkg2)
```shell
# 进入yum的repos目录
cd /etc/yum.repos.d/
# 修改所有的CentOS文件内容
sed -i 's/mirrorlist/#mirrorlist/g' /etc/yum.repos.d/CentOS-*  
sed -i 's|#baseurl=http://mirror.centos.org|baseurl=http://vault.centos.org|g' /etc/yum.repos.d/CentOS-*  
# 先将原BaseOS配置进行备份
mv /etc/yum.repos.d/CentOS-Linux-BaseOS.repo /etc/yum.repos.d/CentOS-Linux-BaseOS.repo.bak  
# 再下载新配置
wget -O /etc/yum.repos.d/CentOS-Linux-BaseOS.repo http://mirrors.aliyun.com/repo/Centos-vault-8.5.2111.repo
```
2. 清空缓存
```shell
sudo yum clean all
sudo yum makecache
```

## Linux8安装Cockpit
`Cockpit`是`CentOS 8`内置的一款基于Web的可视化管理工具，对一些常见的命令行管理操作都有界面支持，比如用户管理、防火墙管理、服务器资源监控等，使用非常方便。

1. `CentOS 8`默认已安装Cockpit，直接启动服务即可  
```shell
# 配置cockpit服务开机自启
systemctl enable --now cockpit.socket
# 启动cockpit服务
systemctl start cockpit
```

2.`CentOS 7`上如果要使用Cockpit的话，需要自行安装，并开放对应服务  
```shell
# 安装
yum install cockpit
# 开放服务
firewall-cmd --permanent --zone=public --add-service=cockpit
# 重新加载防护墙
firewall-cmd --reload
```

3. 安装完成后即可通过浏览器访问Cockpit，使用Linux用户即可登录（比如root用户）  
_访问地址：http://ip地址:9090_

## Linux配置Maven
1. 下载 [maven](http://maven.apache.org/download.cgi)
2. 配置环境变量
> vim /etc/profile

```shell
export MAVEN_HOME=/usr/local/apache-maven-3.6.1
export PATH=$MAVEN_HOME/bin:$PATH 
```


## Linux安装RabbitMQ

### 版本选择

> RabbitMQ 依赖于 erlang 环境，所以先安装 [Erlang](https://www.erlang.org/downloads) 注意二者之间的版本依赖，
>
> 注意 [RabbitMQ](https://www.rabbitmq.com/) 看清楚当前 RabbitMQ 依赖于的 relang [对应版本信息](https://www.rabbitmq.com/which-erlang.html) 。

![image-20211214214253983](../../document/images/public/image-20211214214253983.png)

![image-20211214214448037](../../document/images/public/image-20211214214448037.png)

![image-20211214214642865](../../document/images/public/image-20211214214642865.png)

![image-20211214215101156](../../document/images/public/image-20211214215101156.png)

<u>推荐安装方式:**YUM库安装**，去掉了一些运行RabbitMQ不需要的Erlang模块和依赖项</u>

![image-20211219011132132](../../document/images/public/image-20211219011132132.png)

### 安装：

##### 方式一：yum安装

①下载[Erlang源](https://github.com/rabbitmq/erlang-rpm/releases) 、[RabbitMQ](https://github.com/rabbitmq/rabbitmq-server/releases)  上传到服务器

*注意:CentOS7应该下载el7，CentOS8下载的是el8命名的*

②安装源
> yum localinstall xxxxx.noarch.rpm

### 命令
```shell
#安装页面管理插件
rabbitmq-plugins enable rabbitmq_management
#启动
systemctl start rabbitmq-server
#开机自启
systemctl enable rabbitmq-server
#检查状态
systemctl status rabbitmq-server
#创建用户
rabbitmqctl add_user 用户名 密码
##设置用户角色:  administrator (系统管理员)
rabbitmqctl set_user_tags 用户名 administrator
##设置用户权限： .* 表示最高权限/所有权限
rabbitmqctl set_permissions -p "/" 用户名 ".*" ".*" ".*"
```

## Linux 安装 Mysql

### yum安装Mysql

#### 1、下载[MySQL安装源](https://dev.mysql.com/downloads/repo/yum/)
* 查看系统版本，选择对应的版本进行下载
```shell
#查看Linux版本
cat /etc/redhat-release
## Linux版本：CentOS Linux release 7.6.1810 (Core)
#下载
wget https://dev.mysql.com/get/mysql80-community-release-el7-3.noarch.rpm
```

#### 2、安装源
> yum localinstall mysql80-community-release-el7-3.noarch.rpm

检查是否安装成功：  
在/etc/yum.repos.d/目录下生成两个repo文件：**mysql-community.repo** 及 **mysql-community-source.repo**   
并且通过yum repolist可以看到mysql相关资源

```shell
yum repolist enabled | grep "mysql.*-community.*"
#!mysql-connectors-community/x86_64 MySQL Connectors Community                108
#!mysql-tools-community/x86_64      MySQL Tools Community                      90
#!mysql80-community/x86_64          MySQL 8.0 Community Server                113
```
#### 3、选择MySQL版本
使用MySQL Yum Repository安装MySQL，默认会选择当前最新的稳定版本，  

```shell
#查看当前MySQL Yum Repository中所有MySQL版本（每个版本在不同的子仓库中）
yum repolist all | grep mysql

#切换版本
yum-config-manager --disable mysql80-community
yum-config-manager --enable mysql57-community

```

**注意**   
除了使用*yum-config-manager*之外，还可以直接编辑*/etc/yum.repos.d/mysql-community.repo*文件  
enabled=0禁用，enabled=1启用
```shell
#Disable to use MySQL 5.7
[mysql80-community]
name=MySQL 8.0 Community Server
baseurl=http://repo.mysql.com/yum/mysql-8.0-community/el/7/$basearch/
enabled=0
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql

# Enable to use MySQL 5.7
[mysql57-community]
name=MySQL 5.7 Community Server
baseurl=http://repo.mysql.com/yum/mysql-5.7-community/el/7/$basearch/
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-mysql
```

* 检查当前启用的MySQL仓库
> yum repolist enabled | grep mysql

#### 4、安装MySQL
> yum install mysql-community-server

`出现报错Error:Unable to find a match: mysql-community-server`  
`解决方法：先禁用本地的 MySQL 模块在安装即可`
```shell
yum module disable mysql
yum -y install mysql-community-server
```

#### 5、启动MySQL服务
> systemctl start mysqld

#### 6、配置开机启动
```shell
systemctl enable mysqld
systemctl daemon-reload        ##重新加载服务配置
```

#### 7、修改配置

> vim  /etc/my.cnf

```shell
#①默认策略，使其支持简单密码的设定（可选）；
[mysqld] 节点最后添加：
validate_password=OFF
#②配置默认编码（可选）：
[client]
default-character-set = utf8mb4
[mysqld]
......
character-set-client-handshake = FALSE
character-set-server = utf8mb4
collation-server = utf8mb4_unicode_ci
init_connect='SET NAMES utf8mb4'
```

*备注：查看编码
> SHOW VARIABLES WHERE Variable_name LIKE 'character_set_%' OR Variable_name LIKE 'collation%';

	系统变量                         描述
	character_set_client    (客户端来源数据使用的字符集)
	character_set_connection    (连接层字符集)
	character_set_database    (当前选中数据库的默认字符集)
	character_set_results    (查询结果字符集)
	character_set_server    (默认的内部操作字符集)
	这几个变量必须是utf8mb4。

**保存并重启MySQL**

#### 8、初始化密码
> grep 'temporary password' /var/log/mysqld.log

_执行命令结果示例如下:_
*2020-04-08T08:12:07.893939Z 1 [Note] A temporary password is generated for root@localhost: xvlo1lZs7>uI*

#### 9、对MySQL进行安全性配置
> mysql_secure_installation

```shell
#①重置root用户的密码。
Enter password for user root: #输入上一步获取的root用户初始密码
The 'validate_password' plugin is installed on the server.
The subsequent steps will run with the existing configuration of the plugin.
Using existing password for root.
Estimated strength of the password: 100
Change the password for root ? ((Press y|Y for Yes, any other key for No) : Y #是否更改root用户密码，输入Y
New password: #输入新密码，长度为8至30个字符，必须同时包含大小写英文字母、数字和特殊符号。特殊符号可以是()` ~!@#$%^&*-+=|{}[]:;‘<>,.?/
Re-enter new password: #再次输入新密码
Estimated strength of the password: 100
Do you wish to continue with the password provided?(Press y|Y for Yes, any other key for No) : Y #是否继续操作，输入Y

#②删除匿名用户账号。
By default, a MySQL installation has an anonymous user, allowing anyone to log into MySQL without having to have a user account created for them. This is intended only for testing, and to make the installation go a bit smoother. You should remove them before moving into a production environment.
Remove anonymous users? (Press y|Y for Yes, any other key for No) : Y  #是否删除匿名用户，输入Y
Success.

#③禁止root账号远程登录。
Disallow root login remotely? (Press y|Y for Yes, any other key for No) : Y #禁止root远程登录，输入Y
Success.

#④删除test库以及对test库的访问权限。
Remove test database and access to it? (Press y|Y for Yes, any other key for No) : Y #是否删除test库和对它的访问权限，输入Y
- Dropping test database...
  Success.

#⑤重新加载授权表。
Reload privilege tables now? (Press y|Y for Yes, any other key for No) : Y #是否重新加载授权表，输入Y
Success.
All done!
```

#### 10、创建新用户
①输入root用户的密码登录MySQL
> mysql -uroot -p

②grant创建并授权新用户
```shell
mysql> grant all on *.* to 'dms'@'%' IDENTIFIED BY '123456';
mysql> flush privileges;
```

### Mysql命令
```shell
#启动  
sudo systemctl start mysqld.service  
#停止  
sudo systemctl stop mysqld.service
#重启
sudo systemctl restart mysqld.service
#查看状态
sudo systemctl status mysqld.service
```

## Linux安装Git

### 查看当前版本
> git --version

### git卸载
> yum remove git

### 源码编译安装

1. 下载解压[git版本](https://github.com/git/git/releases)
2. 安装依赖
```shell
cd git-xxx
yum install curl-devel expat-devel gettext-devel openssl-devel zlib-devel gcc perl-ExtUtils-MakeMaker
```
`依赖安装完后，由于默认安装了git，需要删除：yum remove git`

4. 编译源码：
> make prefix=/usr/local/git all
5. 安装
> make prefix=/usr/local/git install
6. 配置环境变量
> vim /etc/profile
```shell
#添加git相关配置：
PATH=$PATH:/usr/local/git/bin
export PATH
```
* 刷新环境变量
> source /etc/profile

## 安装GCC编译器
### 安装
1. 查看当前gcc版本**   
```shell
gcc --version
```
3. 如果当前未安装，先安装默认版本
``` shell
yum install -y glibc-static libstdc++-static
yum install -y gcc gcc-c++
```
* `第一行指令用于安装编译 C 和 C++ 代码所需的静态链接库;`  
* `第二行指令用于安装编译 C 和 C++ 代码的 gcc 和 g++ 指令`

3. 下载指定版本的 [gcc](http://ftp.tsukuba.wide.ad.jp/software/gcc/releases/) ([官网](https://gcc.gnu.org))上传到指定路径（usr/local）,并解压

4. 进入解压后的文件，安装GCC所需依赖包(依赖了mpfr、gmp、mpc 和isl共四个库)  
`确定安装bzip2，使用如下命令安装bzip2：yum -y install bzip2 `  

> cd /usr/local/gcc-xxx  

> ./contrib/download_prerequisites

5. 安装GCC编译器前提工作
```shell
#①手动创建一个目录，用于存放编译 GCC 源码包生成的文件（usr/local目录下）
mkdir gcc-build-xxx  
cd gcc-build-xxx 
#②必要配置，使其支持编译 C 和 C++ 语言：
../gcc-xxx/configure --enable-checking=release --enable-languages=c,c++ --disable-multilib
```
6. 使用 make 命令来编译 GCC 源程序  
`前提：执行命令：yum -y install texinfo`  
> make  

`注意，编译过程是非常耗时的（可能耗时 5-6 小时完成编译）`  

7. 安装 gcc  
```shell
make install
```
9. 重启系统

### 更新GCC版本动态库
1、<font color='red'>问题分析</font>：

> ```
> 源码编译升级安装了gcc后，编译程序或运行其它程序时，有时会出现类似:
> /usr/lib64/libstdc++.so.6: version `GLIBCXX_3.4.21' not found 的问题。
> 这是因为升级gcc时，生成的动态库没有替换老版本gcc的动态库导致的，将gcc最新版本的动态库替换系统中老版本的动态库即可解决
> ```

2、<font color='red'>查看当前动态库</font>：

> ```shell
> strings /usr/lib64/libstdc++.so.6 | grep GLIBC
> ```

输出结果：

> ```shell
> GLIBCXX_3.4
> GLIBCXX_3.4.1
> GLIBCXX_3.4.2
> GLIBCXX_3.4.3
> GLIBCXX_3.4.4
> GLIBCXX_3.4.5
> GLIBCXX_3.4.6
> GLIBCXX_3.4.7
> GLIBCXX_3.4.8
> GLIBCXX_3.4.9
> GLIBCXX_3.4.10
> GLIBCXX_3.4.11
> GLIBCXX_3.4.12
> GLIBCXX_3.4.13
> GLIBCXX_FORCE_NEW
> GLIBCXX_DEBUG_MESSAGE_LENGTH
> ```

从以上输出可以看出，`gcc`的动态库还是旧版本的。说明出现这些问题，是因为升级`gcc`时，生成的动态库没有替换老版本`gcc`的动态库。

3、<font color='red'>问题处理</font>：

①**查找编译`gcc`时生成的最新动态库**

> ```shell
> find / -name "libstdc++.so*"
> ```

结果：

```shell
/usr/local/gcc-build-11.2/stage1-x86_64-unknown-linux-gnu/libstdc++-v3/src/.libs/libstdc++.so

/usr/local/gcc-build-11.2/stage1-x86_64-unknown-linux-gnu/libstdc++-v3/src/.libs/libstdc++.so.6

/usr/local/gcc-build-11.2/stage1-x86_64-unknown-linux-gnu/libstdc++-v3/src/.libs/libstdc++.so.6.0.21  //最新动态库
……
```

备注：`/usr/local/gcc-build-11.2`是**升级gcc**时的安装目录。

②**将上面的最新动态库`libstdc++.so.6.0.21`复制到`/usr/lib64`目录下**

```shell
 cp /usr/local/gcc-build-11.2/stage1-x86_64-pc-linux-gnu/libstdc++-v3/src/.libs/libstdc++.so.6.0.29   /usr/lib64
```

③**修改系统默认动态库的指向，即：重建默认库的软连接**

切换工作目录至`/usr/lib64`：

```shell
cd /usr/lib64
```

删除原来软连接：

```shell
rm -rf libstdc++.so.6
```

将默认库的软连接指向最新动态库：

```shell
ln -s libstdc++.so.6.0.29 libstdc++.so.6
```

④**升级完成后重新检查动态库**：
```shell
strings /usr/lib64/libstdc++.so.6 | grep GLIBC
```
现在输出如下：
```
···
GLIBCXX_3.4.19
GLIBCXX_3.4.20
GLIBCXX_3.4.21
····
GLIBCXX_3.4.29
GLIBC_2.3
GLIBC_2.2.5
GLIBC_2.3.2
GLIBCXX_FORCE_NEW
GLIBCXX_DEBUG_MESSAGE_LENGTH
```

## jenkins
### 安装
1. 下载   
yum的repos中默认是没有Jenkins的，需要先将Jenkins存储库添加到yum repos。
```shell
wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
#yum安装Jenkins
yum install jenkins
```
2. 修改配置  
① `/etc/sysconfig/jenkins`
```shell
#启动用户修改为root
JENKINS_USER="jenkins" 
#Jenkins默认端口是8080，与tomcat的默认端口冲突，修改一下默认端口
JENKINS_PORT="9083"
```  
② `/etc/init.d/jenkins`
```shell
#修改jdk路径（通过命令:which java 查看jdk路径）：
candidates="
/etc/alternatives/java
/usr/lib/jvm/java-1.8.0/bin/java
/usr/lib/jvm/jre-1.8.0/bin/java
/usr/lib/jvm/java-11.0/bin/java
/usr/lib/jvm/jre-11.0/bin/java
/usr/lib/jvm/java-11-openjdk-amd64
/usr/local/java/jdk1.8.0_271/bin/java
```
**注**：  
高版本的jenkins的配置文件不再是*/etc/sysconfig/jenkins*了，   
而是*/usr/lib/systemd/system/jenkins.service*

**修改用户涉及文件权限**
```shell
chown -R root:root /var/lib/jenkins
chown -R root:root /var/cache/jenkins
chown -R root:root /var/log/jenkins
```
3. 启动
```shell
systemctl daemon-reload  #刷新配置 
systemctl start jenkins.service
```

4. 进入jenkins页面  
Jenkins登录页面: *ip:9083*

### 卸载jenkins
1. rpm卸载
> rpm -e jenkins

2. 检查是否卸载成功
> rpm -ql jenkins

3. 彻底删除残留文件
> find / -iname jenkins | xargs -n 1000 rm -rf

4. 删除文件  

> /etc/sysconfig/jenkins.rpmsave
>
> /etc/init.d/jenkins.rpmsave
>
> /etc/yum.repos.d/jenkins.repo

### jenkins命令
```shell
#查看密码(初始密码): 
tail /var/lib/jenkins/secrets/initialAdminPassword  
#查看jenkins状态
systemctl status jenkins.service   
#关闭
systemctl stop jenkins.service  
#启动
systemctl start jenkins.service             
#设置jenkins开机启动(默认开机启动)：
systemctl enable jenkins.service 
```

### jenkins插件
```text
* 用户权限：Role-based Authorization Strategy  
* 全局安全配置,授权策略    Role-Based Strategy
```

## Linux安装redis
### 安装
1. 下载[redis](https://redis.io/download), 解压  
2. 安装  
```shell
#①cd 进入解压后文件根目录下，执行命令编译,安装
make
make install PREFIX=/usr/local/redis
#②移动redis配置文件到redis/bin/etc文件夹下：
mkdir /usr/local/redis/etc
mv redis.conf /usr/local/redis/etc
#③创建文件目录 
mkdir /usr/local/redis/data  /usr/local/redis/log	/usr/local/redis/run
```

3. 修改配置文件  
>vim redis.conf

```shell
#①将daemonize属性改为yes（表明需要在后台运行）
#②绑定的主机地址
bind 172.17.233.117 127.0.0.1
#③设置Redis连接密码
requirepass foobared
#③指定本地数据库存放目录
dir /usr/local/redis/data
#④修改日志路径
logfile /usr/local/redis/log/redis.log
#⑤当Redis以守护进程方式运行时,Redis默认会把pid写入/var/run/redis.pid文件，可以通过pidfile指定
pidfile /usr/local/redis/run/redis_6379.pid
```
4. 修改redis以普通用户启动  
   ①创建用户redis
> useradd -s /bash/false -M redis

   ②修改文件拥有者  
> chown -R  User:Group   File

   ③修改文件权限  
> chmod -R 700  File

```text
-rw------- (600)      只有拥有者有读写权限。
-rw-r--r-- (644)      只有拥有者有读写权限；而属组用户和其他用户只有读权限。
-rwx------ (700)     只有拥有者有读、写、执行权限。
-rwxr-xr-x (755)    拥有者有读、写、执行权限；而属组用户和其他用户只有读、执行权限。
-rwx--x--x (711)    拥有者有读、写、执行权限；而属组用户和其他用户只有执行权限。
-rw-rw-rw- (666)   所有用户都有文件读、写权限。
-rwxrwxrwx (777)  所有用户都有读、写、执行权限。
```

5. 配置环境变量  
> vi /etc/profile

```shell
#redis
export REDIS_HOME=/usr/local/redis
export PATH=$PATH:$REDIS_HOME/bin
```

### redis开机自启

**方式一**
1. 创建Redis启动服务  
   在系统开机启动项目录 `/lib/systemd/system` 目录添加 `redis.service` 文件

> vim /lib/systemd/system/redis.service

```shell
[Unit]
Description=The redis-server Process Manager
Documentation=https://redis.io/
After=network.target

[Service]
User=redis
Group=redis
Type=forking
ExecStart=/usr/local/redis/bin/redis-server /usr/local/redis/etc/redis.conf
ExecStop=/usr/local/redis/bin/redis-cli shutdown
Restart=always
#LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
```
2. 设置开机运行  
>systemctl enable redis

**方式二**
> vim /etc/init.d/redis

```shell
#!/bin/bash
#chkconfig: 22345 10 90
#description: Start and Stop redis
REDISPORT=6379
EXEC=/usr/local/redis/bin/redis-server
CLIEXEC=/usr/local/redis/bin/redis-cli
PIDFILE=/var/run/redis.pid
CONF="/usr/local/redis/etc/redis.conf"
case "$1" in
    start)
        if [ -f $PIDFILE ];then
            echo "$PIDFILE exists,process is already running or crashed"
        else
            echo "Starting Redis server..."
            $EXEC $CONF
        fi
        ;;
    stop)
        if [ ! -f $PIDFILE ];then
            echo "$PIDFILE does not exist,process is not running"
        else
            PID=$(cat $PIDFILE)
            echo "Stopping..."
            $CLIEXEC -p $REDISPORT shutdown
            while [ -x /proc/${PID} ]
                do
                    echo "Waiting for Redis to shutdown..."
                    sleep 1
                done
                echo "Redis stopped"
        fi
        ;;
    restart)
        "$0" stop
        sleep 3
        "$0" start
        ;;
    *)
        echo "Please use start or stop or restart as first argument"
        ;;
esac
```
**为启动脚本增加执行权限**
>chmod +x /etc/init.d/redis

**配置开机启动es**  
> chkconfig --add redis

### redis命令
```shell
# 启动 
systemctl start redis
# 停止
systemctl stop redis
# 查看redis运行状态
systemctl status redis
#重启
systemctl restart redis.service
```


## elasticsearch
### 安装
1. 下载 [es](https://www.elastic.co/cn/downloads/elasticsearch)
2. 修改配置信息
```shell
    cluster.name: elasticsearch
    node.name: node-1
    node.attr.rack: r1
    node.master: true
    path.data: /usr/local/elasticsearch-7.12.0/data
    path.logs: /usr/local/elasticsearch-7.12.0/logs
    
    network.host: 172.17.233.117
    http.port: 9200
    cluster.initial_master_nodes: ["node-1"]
    http.cors.enabled: true
    http.cors.allow-origin: "*"
    http.cors.allow-credentials: true  
```
2. 启动es  
> cd /usr/local/elasticsearch-7.12.0  
>
> ./bin/elasticsearch


`常见问题`  
* jdk版本问题  
`future versions of Elasticsearch will require Java 11; your Java version from [/usr/local/nlp/java/jdk1.8.0_162/jre] does not meet this requirement`     
原因：elastic 到7.0之后 默认要使用JDK11以上  
解决：修改为es自带的jdk:    
> vim bin/elasticsearch
```shell
#配置为elasticsearch自带jdk
export ES_JAVA_HOME=/usr/local/elasticsearch-7.15.2/jdk
export PATH=$ES_JAVA_HOME/bin:$PATH
#添加jdk判断
if [ -x "$ES_JAVA_HOME/bin/java" ]; then
JAVA="/usr/local/elasticsearch-7.15.2/jdk/bin/java"
else
JAVA=`which java`
fi
```
* 启动用户  
`es不允许root用户启动，需要新建用户`  
```shell
	#  添加一个用户组
	groupadd XXX
	# 添加一个用户，-g是在用户组下 -p是密码
	useradd elk -g 用户组名 -p 密码
	# 进入es的安装目录
	cd /usr/local/elasticsearch 
	# 给用户XXX授权
	chown -R 用户名:用户组 elasticsearch-5.1.1/
```

* 启动可能出现的问题
```
  [1]: max file descriptors [4096] for elasticsearch process is too low, increase to at least [65535]
	解决：vim /etc/security/limits.conf文件，添加以以下两行即可解决
	* soft nofile 65535
	* hard nofile 65535
  [2]: max number of threads [3818] for user [admin] is too low, increase to at least [4096]
	解决：vim /etc/security/limits.conf文件，添加以下两行即可解决
		* soft nproc  4096
		* hard nproc  4096
  [3]: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
	解决：vim /etc/sysctl.conf 文件，添加以下一行即可解决，并执行 sysctl -p
		vm.max_map_count=655360  
  [4]修改运行内存
  OPENJDK 64-BIT SERVER VM WARNING: INFO: OS::COMMIT_MEMORY(0X00000000C5330000, 986513408, 0) FAILED；
 解决： [root@izwz9eu3mkqq1njlkrfhc8z ~]# vim /usr/local/elasticsearch-6.3.2/config/jvm.options 添加内容
    ## IMPORTANT: JVM heap size
    -Xms256m
    -Xmx256m
```

### es开机自启

1. **创建脚本**  
>vim /etc/init.d/elasticsearch

```shell
#!/bin/sh
#chkconfig: 2345 80 05
#description: elasticsearch

export ES_HOME=/usr/local/elasticsearch-7.15.2/

case "$1" in
start)
    #es的启动账号名
    su uelasticsearch<<!
    #es的安装路径
    cd $ES_HOME
    ./bin/elasticsearch -d -p pid
    exit
!
    echo "elasticsearch startup"
    ;;
stop)
    es_pid=`cat $ES_HOME/pid`
    kill -9 $es_pid
    echo "elasticsearch stopped"
    ;;
restart)
    es_pid=`cat $ES_HOME/pid`
    kill -9 $es_pid
    echo "elasticsearch stopped"
    su uelasticsearch<<!
    cd $ES_HOME
    ./bin/elasticsearch -d -p pid
    exit
!
    echo "elasticsearch startup"
    ;;
*)
    echo "start|stop|restart"
    ;;
esac

exit $?

```
2. **为启动脚本增加执行权限**
> chmod +x /etc/init.d/elasticsearch

3. **配置开机启动es**
> chkconfig --add elasticsearch

### 安装es-head插件

#### 1)安装node
1. 下载 **[node](http://nodejs.cn/download/)** ,解压
2. 配置环境变量 vi /etc/profile
```shell
#node
export NODE_HOME=/usr/local/node-v14.16.1-linux-x64
export PATH=$PATH:$NODE_HOME/bin
export NODE_PATH=$NODE_HOME/lib/node_modules
```
3. 刷新环境变量 
> source /etc/profile

#### 2)安装head
1. 下载 **[elasticsearch-head](https://github.com/mobz/elasticsearch-head)**
```shell
git clone git://github.com/mobz/elasticsearch-head.git
cd elasticsearch-head
npm install -g grunt-cli
npm install
```
`如果npm install一直卡在`Receiving...`不动的话，是因为访问的npm registry网络不行，我们可以修改为淘宝的仓库`  
```shell
#修改为淘宝的源
npm config set registry http://registry.npm.taobao.org
```
2. 修改Gruntfile.js文件
>   cd /usr/local/elasticsearch-head/    
>   vim ./Gruntfile.js

![Gruntfile.js](../../document/images/public/es修改配置Gruntfile.png)

3. 修改elasticsearch-head默认连接地址
> cd /usr/local/elasticsearch-head/_site/  
> vim app.js

搜索 “/this.base_uri” 修改为  
```this.base_uri = this.config.base_uri || this.prefs.get("app-base_uri") || "http://es`s IP:9200"; ```
![app.js](../../document/images/public/es-head文件app.png)

4. 修改elasticsearch服务配置文件允许跨域（在elasticsearch.yml文件中添加）
```shell
##
http.cors.enabled: true
http.cors.allow-origin: "*"
http.cors.allow-credentials: true
```
5. 启动elasticsearch-head
> nohup /usr/local/elasticsearch-head/node_modules/grunt/bin/grunt server & exit

#### elasticsearch-head开机启动
1. **启动脚本**
> vim /etc/init.d/elasticsearch-head

```shell
#!/bin/sh
#chkconfig: 2345 80 05
#description: elasticsearch-head
# nodejs 安装的路径
export NODE_PATH=/usr/local/node-v14.16.1-linux-x64
export PATH=$PATH:$NODE_PATH/bin
# elasticsearch-head的路径
cd /usr/local/elasticsearch-head
nohup npm run start >/usr/local/elasticsearch-head/nohup.out 2>&1 &
```

2. **为启动脚本增加执行权限**
> chmod +x /etc/init.d/elasticsearch-head

3. **配置开机启动es**
> chkconfig --add elasticsearch-head

### elasticsearch 中文分词插件 IK
1. github 下载es对应版本 [elasticsearch-analysis-ik](https://github.com/medcl/elasticsearch-analysis-ik/releases)
2. 安装([HELP](https://github.com/medcl/elasticsearch-analysis-ik))
```shell
#1、use elasticsearch-plugin to install ( supported from version v5.5.1 ):
#命令： ./bin/elasticsearch-plugin install url
./bin/elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v6.3.0/elasticsearch-analysis-ik-6.3.0.zip
#2.restart elasticsearch
```

### elasticsearch pinyin分词器插件 elasticsearch-analysis-pinyin
1. [pinyin插件地址](https://github.com/medcl/elasticsearch-analysis-pinyin/releases)
2. 插件安装
```shell
#命令： ./bin/elasticsearch-plugin install url
./bin/elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-pinyin/releases/download/v7.15.2/elasticsearch-analysis-pinyin-7.15.2.zip
#2.restart elasticsearch
```

## kibana
### 安装
1. 下载[kibana](https://artifacts.elastic.co/downloads/kibana/kibana-7.13.1-linux-x86_64.tar.gz)
2. 修改配置
> vim config/kibana.yml

```shell
server.host: "服务器内网ip"
elasticsearch.url: "http://es服务器ip:9200"
##中文
i18n.locale: "zh-CN"
```
3. 启动
> ./bin/kibana

### 开机自启
1. **启动脚本**
```vim /etc/init.d/kibana```

```shell
#!/bin/bash

# chkconfig:   2345 98  02
# description:  kibana
 
KIBANA_HOME=/usr/local/kibana-7.13.1-linux-x86_64
case $1 in
        start) $KIBANA_HOME/bin/kibana --allow-root &;;
        *) echo "require start";;
esac
```
2. **为启动脚本增加执行权限**  
> chmod +x /etc/init.d/kibana

3. **配置开机启动es**  
> chkconfig --add kibana