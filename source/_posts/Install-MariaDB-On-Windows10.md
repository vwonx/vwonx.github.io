---
title: win10下安装MariaDB
date: 2018-12-01 10:21:39
tags: ["Database","MariaDB"]
categories: "数据库"
---
发现大多数博客，在windows10上安装MariaDB都需要自行拷贝安装目录下的配置文件，然而MariaDB-10.3.11的安装目录下并不存在原先版本的配置文件。故根据官网的说明进行了安装，本文记录安装的过程。

<!--more-->

## 下载MariaDB
1. 进入MariaDB的官网 https://downloads.mariadb.org/
![1.png](https://i.loli.net/2018/12/01/5c01f25a8846f.png)
2. 选择符合系统的版本，这里我选择64位的zip免安装版本
![2.png](https://i.loli.net/2018/12/01/5c01f30353751.png)
3. 解压下载的zip压缩包，并把解压出来的文件夹放在你需要安装的位置
![3.png](https://i.loli.net/2018/12/01/5c01f385c8cf7.png)

## 根据官方文档进行安装
安装使用的是bin目录下的mysql_install_db.exe，以下是官方的介绍 https://mariadb.com/kb/en/library/mysql_install_dbexe/
>The functionality of mysql_install_db.exe is comparable with the shell script mysql_install_db used on Unix, however it has been extended with both Windows specific functionality (creating a Windows service) and to generally useful functionality. For example, it can set the 'root' user password during database creation. It also creates the my.ini configuration file in the data directory and adds most important parameters to it (e.g port).

>mysql_install_db.exe is used by the MariaDB installer for Windows if the "Database instance" feature is selected. It obsoletes similar utilities and scripts that were used in the past such as mysqld.exe --install, mysql_install_db.pl, and mysql_secure_installation.pl.

可以看出可以使用它来进行快速安装，配置root密码，并在数据文件夹中生成 my.ini 配置文件。

1. 运行cmd或powershell，进入MariaDB安装目录下的bin文件夹
![4.png](https://i.loli.net/2018/12/01/5c01f5ee401e8.png)
<font color="red">Note：为了避免安装过程中权限的错误，需使用管理员运行</font>
2. 使用mysql_install_db.exe进行初始化设置
   ```bash
   mysql_install_db.exe --datadir=path --service=name --password=secret
   ```
   ![5.png](https://i.loli.net/2018/12/01/5c01f8af3f5eb.png)
   datadir：数据文件夹地址，之后新建数据库都在此文件夹中
   
   service：windows服务名字

   password：root账号的密码

   执行成功后，在数据文件夹中也建立了 my.ini 文件
   ![6.png](https://i.loli.net/2018/12/01/5c01f9b0be962.png)

   其他的参数具体可以下图
   ![11.png](https://i.loli.net/2018/12/01/5c01fe3be2ee8.png)

3. 添加bin文件夹到环境变量中
   ![7.png](https://i.loli.net/2018/12/01/5c01fafebbf1f.png)

## 启动服务
1. 如果之前安装的时候，有使用service参数，可以使用下面命令进行启动
   ```bash
   net start 你之前取得名字
   ```
   <font color="red">Note：需使用管理员运行</font>

   由于我取得就是MariaDB，所以使用MariaDB
   ![8.png](https://i.loli.net/2018/12/01/5c01fc6cb7c65.png)

2. 可以使用windows的任务管理器进行启动
   ![9.png](https://i.loli.net/2018/12/01/5c01fcb9c6059.png)
   右键启动即可

## 登录MariaDB
   ```bash
   mysql -u root -p
   ```
   敲入之前设置的密码，就可正常登录
   ![10.png](https://i.loli.net/2018/12/01/5c01fdb0b4237.png)

