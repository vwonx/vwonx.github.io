---
title: Ubuntu 20.04 Server + Virtual Box安装过程问题记录与解决
subtitle: 记录使用virtual box安装ubuntu 20.04 server过程中发生的问题及解决过程
catalog: true
tags:
  - Linux
categories:
  - Linux
abbrlink: '433e5440'
date: 2022-05-09 12:19:44
---

由于习惯了使用云开发机进行项目的开发，因此为了模拟云开发机的环境，我通常会使用虚拟机安装一个Linux系统作为云开发机，并利用虚拟机克隆机制做备份、多开发环境部署等操作。

近日，原先虚拟机上安装的CentOS 8发现无法通过yum包管理来安装软件。后经查询，发现CentOS 8已经结束了生命周期（EOL），社区已不再维护。所以如果需要使用yum包来下载软件，需要更新源。（例如替换为阿里源）

出于上述原因，再加考虑到好像ubuntu 20.04的版本自己没有用过，所以打算直接将旧的CentOS替换为Ubuntu。我一般的步骤如下：
1. 先安装一台虚拟机（命名为A），并在安装完成后进行部分通用型软件的安装，如 “net-tools”。
2. 在虚拟机A安装好通用软件后，将该状态创建一个快照。
3. 以步骤2中的快照作为蓝本，后续全部的开发机以这个蓝本为基础进行克隆，并部署对应所需的环境。

上述三个步骤可以保证，就算我开发机的环境崩溃了，我也不需要重新安装一遍系统。而是可以直接再以快照作为蓝本克隆出一个新的虚拟机作为开发机。

接着，要解决的另一个问题，就是虚拟机IP分配的问题。由于virtual box默认的网卡使用的是**NAT**模式，该模式的特点是简单，不需要任何配置，就可以使虚拟机连上互联网。但是弊端是ip是动态的，会发生变化。而我的虚拟机是作为开发机使用的，平常均是vscode+ssh插件实现与开发机之间的连接。因此，这种会发生变化的ip对于作为开发机的虚拟机来说是极为不便的。

为了解决上述问题，我的做法通常是给虚拟机再加一块网卡，并设置为**Host-only**模式，这样后续的ip都会是固定的。这种做法在使用CentOS作为系统的时候，没发生过任何问题，即在克隆快照后也会默认分配一个新的ip地址。但是在换到ubuntu后，依次发现了两个问题。下面对问题进行记录，并附上解决方法。

# 问题一：系统找不到第二块Host-only模式的网卡

## 问题描述
在安装完系统后，添加第二块**Host-only**模式的显卡。在进入系统后，发现依旧只有旧的**NAT**模式网卡。

## 问题复现
经过多次系统安装后发现，如果是在给虚拟机安装系统前就添加了网卡，那么在安装完成后不会出现该问题。该问题发生的情况是在安装完系统后，再添加网卡。

## 问题定位
在系统里查看网络设备（执行 ipconfig -a），发现确实有这个网卡，但是ip地址是空的。想到是否可能是该网络没有激活。在执行下述命令临时启动网卡并进行ip分配后，发现问题解决，由此问题定位完成。
```bash
ifconfig -a # 这一步仅是查看并记录没有开启的网卡的编号
ifconfig 上一步记录的网卡编号（如enp0s8） up # 启动网卡
dhclient 上一步记录的网卡编号 # 分配ip
```

## 问题解决方法
上述问题定位里的解决方案可以临时解决ip问题，但是并不是永久配置，会在系统重启后会再次失效。永久配置的方法如下：
```bash
vim /etc/netplan/00-installer-config.yaml # 打开netplan的yaml文件，可能名字会不一样
```
通常而言，如果在安装系统的过程中没有进行手动ip分配，那么yaml文件的内容大致如下：
```yaml
# This is the network config written by 'subiquity'
network:
  ethernets:
    enp0s3:
      dhcp4: true
  version: 2
```
很明显可以看到，他并没有配置enp0s8编号的网卡，因此我们只需要将它补上，也直接使用DHCP协议自动分配ip就行。修改后的文件如下：
```yaml
# This is the network config written by 'subiquity'
network:
  ethernets:
    enp0s3:
      dhcp4: true
    enp0s8:
      dhcp4: true
  version: 2
```
之后重启系统，问题解决。

# 问题二：克隆快照得到的虚拟机ip地址相同
## 问题描述
在进行快照克隆后，本来得到的虚拟机的Host-only网卡应该会分配一个新的ip地址（CentOS是可以的），但是后来发现得到的是一样的ip地址。

## 问题复现
对快照进行克隆，就可以复现该问题。

## 问题定位
这个问题定位花了比较多的时间，网上一些博客说需要换网卡的MAC地址。但这个显然不行，因为现在版本的virtual box对虚拟机克隆的时候会自动随机分配一个MAC地址。为了验证这个想法，也进行了实验。结果果然如预料一般。那么显然DHCP协议分配ip的时候并不是看MAC地址，那么会不会是以其他的编码作为id呢？抱着这个想法在外网上查找了一番后，见到了如下的一种[说法](https://forums.virtualbox.org/viewtopic.php?f=6&t=100850#p489404)：
![1](https://raw.githubusercontent.com/vwonx/blog-imgs/master/ubuntu-vb-problems-record/1.png)
他指出除了MAC地址外，DUID也被用作DHCP的唯一识别id。主要是应对当下MAC地址随机趋势和一些个人隐私安全问题。而Linux系统有些使用machine id作为DUID。看到这里，就明白为什么重新随机MAC地址也无法改变IP地址了。

## 问题解决方法 
知道了原因，那么解决方法就很简单了，只要删除克隆得到的虚拟机的原machine id并进行重新生成即可。命令如下：
```bash
rm /etc/machine-id
systemd-machine-id-setup
```