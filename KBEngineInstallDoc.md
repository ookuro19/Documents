# 1. 源码获取与编译

```
[root @ localhost ~]# yum -y install openssl-devel (在Ubuntu类系统上, 使用 "apt-get install libssl-dev")
[root @ localhost ~]# yum -y install mysql-server (在Ubuntu类系统上, 使用 "apt-get install mysql-server")
[root @ localhost ~]# yum -y install mysql-devel (在Ubuntu类系统上, 使用 "apt-get install libmysql-dev")
[root @ localhost ~]# yum -y install gcc+ gcc-c++(在Ubuntu类系统上, 使用 "apt-get install gcc
下载源码包：
root @ localhost ~]# wget -c https://github.com/kbengine/kbengine/archive/v2.4.2.zip (tar的话为 wget -c https://github.com/kbengine/kbengine/archive/v2.4.2.tar.gz)
root @ localhost ~]# unzip v2.4.2.zip (tar的话为tar -xf v2.4.2.tar.gz)
[root @ localhost ~]# cd kbengine/kbe/src
[root @ localhost/ src]# chmod -R 755 .
[root @ localhost/ src]# make
```

**注意：**
1. 如果出现std的问题，说明g++版本不足，可以参考以下链接，确实的依赖包手动安装即可，下载好的相应文件不要删除
http://www.openskill.cn/article/372
``` 

由于gcc 4.8.2不支持C++11的regex库，故需升到4.9.2
首先加载源，导入rpm

centos6系列
# wget https://www.softwarecollections.org/repos/rhscl/devtoolset-3/epel-6-x86_64/noarch/rhscl-devtoolset-3-epel-6-x86_64-1-2.noarch.rpm
# rpm -ivh rhscl-devtoolset-3-epel-6-x86_64-1-2.noarch.rpm
# wget https://copr.fedoraproject.org/coprs/rhscl/devtoolset-3/repo/epel-6/rhscl-devtoolset-3-epel-6.repo && mv ./*.repo /etc/yum.repos.d/

**centos7系统**
# wget https://www.softwarecollections.org/repos/rhscl/devtoolset-3/epel-7-x86_64/noarch/rhscl-devtoolset-3-epel-7-x86_64-1-2.noarch.rpm
# rpm -ivh rhscl-devtoolset-3-epel-7-x86_64-1-2.noarch.rpm

可能缺少以下组件
# yum -y install libgfortran policycoreutils-python

开始安装
# yum --disablerepo='*' --enablerepo='rhscl-devtoolset-3' install devtoolset-3-gcc devtoolset-3-gcc-c++ devtoolset-3-toolchain -y

安装完成后，启用开发环境：
# scl enable devtoolset-3 bash
这样不会破会你之前系统依赖的GCC环境！
 
设置全局变量，启用新Gcc版本，编译软件
# export CC=/opt/rh/devtoolset-3/root/usr/bin/gcc
# export CPP=/opt/rh/devtoolset-3/root/usr/bin/cpp
# export CXX=/opt/rh/devtoolset-3/root/usr/bin/c++

```
2. 如出现下面字样，说明autoconf未安装，yum安装即可
```
./autogen.sh: 5: ./autogen.sh: autoconf: not found
```
3. autoconf版本较低时，可以参考如下链接
```
https://blog.csdn.net/yongcto/article/details/24791885

wget http://ftp.gnu.org/gnu/autoconf/autoconf-2.68.tar.gz

tar xzf autoconf-2.68.tar.gz

cd autoconf-2.68

./configure

make && make install
```
4. 如出现以下字样，说明libtool未安装或版本较低
```
You need GNU libtoolize 

# yum install -y libtool
``` 

5. unzip都能没装，，，
```
yum install -y unzip zip
```
6. 服务器内存最好在4G，2G的话需要开交换分区，可参考
https://www.jianshu.com/p/d7682a1a5eb9

```
g++: internal compiler error: Killed (program cc1plus)Please submit a full bug report,查了很多资料，最后发现主要原因是内存不足, 临时使用交换分区来解决吧
sudo dd if=/dev/zero of=/swapfile bs=64M count=16
sudo mkswap /swapfile
sudo swapon /swapfile

After compiling, you may wish toCode:

sudo swapoff /swapfile
sudo rm /swapfile
```

# 2. 安装与配置
ComblockEngine的环境安装非常简单，只需要完成MySql的安装和配置即可。MySql的安装和配置也会分Linux和Windows两个平台进行介绍。其他可选的安装和配置，根据实际需求进行选择。

1. 安装MySQL数据库并配置
2. 创建数据库
3. 创建kbe系统用户（可选） 
4. 设置环境变量（可选）
5. Windows平台下，安装Python环境（当需要使用引擎提供的工具时可选）

## 2.1 安装MySQL数据库并配置：
Linux系统下，安装Mysql比较简单，只需执行几个命令即可。
```
安装
[root @ localhost ~]# yum install mysql-server (在Ubuntu类系统上, 使用 apt-get install mysql-server)

设定为开机自动启动
[root @ localhost ~]# chkconfig mysqld on(在Ubuntu类系统上, 安装 sysv-rc-conf)

启动mysql服务
[root @ localhost ~]# /etc/init.d/mysql start (在Ubuntu类系统上, 使用/etc/init.d/mysqld start)

检查是否启动成功
[root@localhost ~]# /etc/init.d/mysqld status
mysqld (pid  9234) is running...

安装成功
```
## 2.2 创建数据库：

首先登入mysql
```
mysql -u root -p
```

>-u 表示选择登陆的用户名， -p 表示登陆的用户密码，上面命令输入之后会提示输入密码，此时输入密码就可以登录到mysql。
### 2.2.1 创建数据库，本章节中假设数据库名为“kbe”
```
mysql> create database kbe;
```
### 2.2.2 删除匿名用户
（一些系统中不删除匿名用户会出现使用kbe账号用本地IP登录mysql被拒绝访问）
```
mysql> use mysql 
mysql> delete from user where user=''; 
mysql> FLUSH PRIVILEGES;
```
### 2.2.3 创建数据库用户
用户名是”kbe”，密码是”pwd123456”（本章节默认使用该账号和密码，请暂时不要修改）
```
mysql> grant all privileges on *.* to kbe@'%' identified by 'pwd123456';
mysql> grant select,insert,update,delete,create,drop on *.* to kbe@'%' identified by 'pwd123456';
mysql> FLUSH PRIVILEGES;
```

### 2.2.4 验证
**Windows系统下**：进入你的mysql安装目录找到mysql.exe(如C:\mysql\bin), 然后在CMD执行如下命令:

```
C:\mysql\bin> mysql -ukbe -ppwd123456 -hlocalhost -P3306
```
**Linux系统下**：

```
[root@localhost ~] mysql -ukbe -ppwd123456 -hlocalhost -P3306
```

如果能成功登陆，说明验证成功！

**注意：**
1: 默认Mysql端口为3306， 如不一致请修改kbengine_defaults.xml->dbmgr->330x

2: 请不要使用其他任何第三方工具来测试，必须使用mysql进行测试，第三方工具为了能够正确的连接到Mysql可能会采用一些兼容的方式，但这对于游戏服务端来说是不可靠的方式，权限应该由用户明确的设置。

## 2.3 创建kbe系统用户（可选）：
创建一个独立的用户来运行KBEngine将会更加安全可靠以及便于维护。如果您跳过此步骤，会使用你当前账户的用户作为kbe user。

**Linux:**
```
[root@localhost ~]# useradd kbe
[root@localhost ~]# passwd kbe

New UNIX password: ******
Retype new UNIX password: ******
passwd: all authentication tokens updated successfully 
```

## 2.4 设置环境变量（可选）
（提醒：在服务端资产库启动服务器的脚本中已经具备自动化临时设置环境变量正确启动服务器功能，因此没有特别复杂的部署需求可略过此步骤。）

CBE引擎会读取系统中设置的(KBE_ROOT, KBE_RES_PATH, KBE_BIN_PATH)环境变量, 环境变量的意义:

**UID**： 操作系统用户账号的uid将被用于区分不同的服务器组，如果是多台硬件服务器共同维护一组服务， 那么每一台机器上的系统uid环境变量都应该保持一致，否则无法形成服务组。 另外uid必须大于0, 小于32767. (注意: Windows系统账号没有UID属性， 需要用户自己添加这个环境变量)

**KBE_ROOT**： 引擎根目录。

**KBE_RES_PATH**： 引擎的资源路径。不同路径使用’:’或者’;’分隔, Windows由于操作系统规则必须使用’;’分隔，默认情况下资源路径中第一个资源路径 是引擎的资源路径， 第二个资源路径是用户脚本的资源路径。

**KBE_BIN_PATH**： 引擎可执行文件所在目录。



**Linux:**
```
(假如kbe被安装在~/目录)
[kbe@localhost ~]$ vim ~/.bashrc

ulimit -c unlimited
export KBE_ROOT=~/kbengine  -> 注意此处最后木有"/",以免之后的路径多一个"/"
export KBE_RES_PATH=$KBE_ROOT/kbe/res/:$KBE_ROOT/assets/:$KBE_ROOT/assets/scripts/:$KBE_ROOT/assets/res/
export KBE_BIN_PATH=$KBE_ROOT/kbe/bin/server/

使环境变量生效:
[kbe@localhost ~]$ source ~/.bashrc

root权限设置用户kbe的uid(假如设置为10103):
[root@localhost ~]# usermod -u 10103 kbe
```

# 3. 创建,启动与关闭
## 3.1 创建资产库
创建资产库需要使用引擎根目录下的`new_assets.sh`，如果修改了环境变量，则可以备份后删除判断KBE_ROOT相关的代码

>**什么是资产库？**
所谓资产库，即是一个新的项目的所有数据、脚本代码、资源等存储的地方，而这个地方就是一个文件夹，该文件夹一般可以放置在引擎根目录下(与kbe、assets文件夹同级)。
>**Tips:**
资产库文件夹也可以放置在其他地方，不过需要修改启动脚本中环境变量，在本文后面会进行阐述。

## 3.2 启动与关闭
默认的，启动引擎的快捷入口是存放在自己项目的资产库下的一个批处理文件start_server.bat(Linux下是start_server.sh)，而引擎在根目录下提供了一个默认的“assets”资产库，该资产库是一个最小的KBE项目。

运行start_server.bat即可启动，而关闭服务器是通过`kill_server.bat`或`safe_kill.bat`。

启动与关闭的执行命令
根据自己所在操作系统不同，有两种方式启动：

**Linux：**
```
（假设引擎安装目录为 root/kbengine）
[kbe @gameserver ~]$ cd root/kbengine/getstarted_assets
[kbe @gameserver ~]$ sh start_server.sh
[kbe @gameserver ~]$ sh kill_server.sh
```

