---
title: GPU服务器管理常见问题
subtitle: 该文对研究生三年管理GPU服务器期间遇到的常见问题进行记录
catalog: true
tags:
  - Linux
  - 教程
categories:
  - Linux
abbrlink: 6f3d798f
date: 2022-05-10 07:23:52
sticky: 1
---

本文主要对管理GPU服务器期间遇到的各种问题提供一种解决方案，希望能帮助到后续管理设备的同学。
注意，各种问题可能存在多种解决方式，下文提供的解决方案，仅在所标注的版本的系统里有效。如后续操作系统版本升级，可能也会出现无法适用的情况。
同样，所提供的解决方案也有可能并不是最佳的，因此更重要的还是提供一种类似的思想以供参考。

# 管理员必知
一些基础的Linux就不再提
## vim 编辑器
vim一定要稍微会用一点，因为有的时候给服务器配置的时候，是没有图形化界面的文本编辑器的。稍微去学一点，知道怎么打开文件、修改文件和保存文件就可以了。
## Linux文件权限
这个属于Linux的基础知识了，稍微网上找一下看一看。做到能看明白一个文件的权限、所有者就行。
## scp 命令
从本地拷贝文件到服务器的命令，我知道很多人用的是windows下的具有图形化界面的终端软件，这些软件可以直接拖拽进行复制。这样的做法是ok的，但是某些情况，是没有这样的软件，甚至本地的环境就是没有UI界面的黑框子。此外，也有些好用的其他拷贝命令，不如rz/sz，但是这些命令不一定系统有装，另外它也需要本地机是图形化界面。综上，scp命令是特殊情况下的一个比较好的解决方案。
scp命令也很简单，只需要掌握两种，一种拷贝文件，一种拷贝文件夹的即可，如下：
```bash
scp 本地文件 服务器上的用户名@服务器IP地址:在服务器上保存的地址
scp -r 本地文件夹 服务器上的用户名@服务器IP地址:在服务器上保存的地址
``` 
## passwd 命令
虽然现在已经使用密钥进行登陆了，但是有些时候需要使用密码，比如用户需要装一个什么软件在他自己家目录下等。或者管理员使用su命令切换账户到其他账户，就需要指定其他账户的密码。所以，当密码忘记时，需要使用passwd命令进行更改
```bash
sudo passwd 需要改密码的账户名
```
## chown 命令
更改文件所有者的命令，简单来说，就是某个文件对于某个用户来说没有权限操控。所以把文件赋予给他，使他可以操控这个文件。
```bash
sudo chown 用户名:用户名 要更改所有者的文件地址

# 有一个情况是需要把一整个文件夹，包括文件夹里面的子文件夹和文件都更改所有者，则用这个命令
sudo chown -R 用户名:用户名 要更改所有者的文件夹地址
```

# 系统安装
## 服务器
推荐Ubuntu 18.04 server（这个系统安装是安装在服务器上的，建议联系当时卖服务器的卖家进行安装。另外注意系统不要装太新的，本文写的时间ubuntu最新的版本为20.04，但是我建议采用老一个版本。原因是如果遇到什么技术问题，网络上可以参考的资料会比较多），自己安装的话，需要注意以下几个点：
1. 网络IP设置问题 （☆☆☆☆☆） 
由于服务器几乎都是走内网的，所以可能IP不是动态分配的，而是采用静态配置的方式。**因此一定要在重装系统前查看一下当前的ip配置，做个备份。在安装完成后，按照之前的配置，再次进行静态分配**（见服务器IP静态分配）。
2. 硬盘挂载问题 （☆☆☆）
服务器的硬盘通常由固态硬盘+机械硬盘组成，固态硬盘一般都是用来安装系统。机械硬盘则由于大容量的特点，几乎都是用来保存数据集的。**安装系统后，硬盘会处于未挂载的情况，需要进行永久挂载**（见服务器硬盘挂载）。
此外，如果是重装系统，机械硬盘的数据一般不会清理掉，会得以保留。但是由于原用户在新装的机子上不存在，因此**重装系统后建立的新用户对他原先存在机械硬盘里的数据文件没有操作权限**，需要管理员进行文件所有者更改后才能使用（见服务器重装后，用户无法操控机械硬盘里原数据文件）。
3. 密钥登陆和防火墙设置问题（☆☆☆）
为了安全，现在服务器全面取消ssh的密码登陆，而改用密钥登陆。因此注意**无论是安装系统后，还是重装系统后，都需要改用密钥登陆并设置防火墙。** 其中密钥登陆这部分涉及的内容比较多，可能遇到的问题也比较多，具体见本文服务器密钥相关一节。

## 普通PC
直接安装Ubuntu任一桌面版本的系统就行了。因为机子都属于个人机，且方便连接显示器操控。所以一般安装完成后直接采用动态ip分配就可以，硬盘挂载也无所谓永久还是临时。至于密码一块，因为都是直接开机使用，不涉及远程ssh连接使用，所以也不用设置密钥登陆。（总而言之，就是个人机由于重装系统等都很方便，很多情况下根据使用者的喜好进行设置就行，管理员不用过多干涉）

# 系统安装完成后所需装的软件
## GCC、G++和make等工具
新版的ubuntu系统已经不会自带这些编译器和相关工具了，但是在后续安装NVIDIA驱动的过程中是需要用到这些工具的。因此安装完系统后，需要将这些工具补齐。执行如下命令：
```bash
sudo apt update
sudo apt install build-essential # 这个包含有GCC、G++和make等工具
```
上述安装完成后，应该就可以继续之后的其他软件安装了。但是难保随着系统和包的变化，有些工具会在未来缺失。因此，如果后续安装其他工具的过程中，提示缺少某些包，只需要把缺少的包再进行安装就好。命令为：
```bash
sudo apt install 缺少的包 # 提示缺少什么，就去搜索下包含这个东西的包叫什么，然后进行安装
```

## 显卡驱动
这类教程很多，可以网上进行搜索，大致流程如下：
1. 去NVIDIA官方选择显卡型号，下载驱动文件。
2. 禁用nouveau驱动
3. 进入纯字符界面
```bash
alt+ctrl+F1
sudo init 3
```
4. 用sudo bash命令安装下载的驱动文件
安装过程中的选项，按照如下进行选择即可：
```
Q:The distribution-provided pre-install script failed! Are you sure you want to continue?
>>> yes
Q:Would you like to register the kernel module souces with DKMS? This will allow DKMS to automatically build a new module, if you install a different kernel later?
>>> No
Q:Would you like to sign the NVIDIA kernel module?
>>> Install without signing
Q:Nvidia’s 32-bit compatibility libraries? 
>>> No
Q:Would you like to run the nvidia-xconfigutility to automatically update your x configuration so that the NVIDIA x driver will be used when you restart x? Any pre-existing x confile will be backed up.
>>> Yes
```

此外，安装过程中也会由可能出现一些报错，如果是提示XX路径找不到，如make之类的，此时请检查是否安装了相应的工具。还有可能出现说c++版本不对，那么请去安装相应版本的c++。总体来说可能报的错误不一定，这里无法列举出全部，如果在安装过程中遇到了错误，请善用各大搜索引擎！这里附上一个可以供参考的[安装过程](https://zhuanlan.zhihu.com/p/463656273)。

## CUDA
1. 查看可以使用的CUDA版本，这里是[地址](https://docs.nvidia.com/cuda/cuda-toolkit-release-notes/index.html)。**注意一点，cuda版本和后续用户安装的Pytorch版本是有关系的，所以不一定就是直接安装最新的。一定要确认下，目前多数用户使用的版本是什么。如果是重装系统，尽量装和之前一样的版本。**
2. 去NVIDA官网下载CUDA。选项分别为Linux、x86_64、ubuntu、系统版本、runfile（local）。
3. 正常用sudo命令安装下载的文件就行
4. 安装过程选项如下：
```
会先有个阅读声明，按D往下翻，然后accept。
接着会出现一个CUDA Installer
第一个选项显卡驱动的‘X’去掉，因为我们已经安装过显卡驱动了
其余分别是 Toolkit、Samples、Demo Suit、Document，默认都选就行
然后选择install
```
5. 环境配置，这里指全局配置。
```bash
sudo vim /etc/profile

# 把下面两行加入文件最后的位置
export PATH=/usr/local/你自己安装的cuda版本/bin${PATH:+:${PATH}} # 如果不确定版本，去/usr/local目录下查看
export LD_LIBRARY_PATH=/usr/local/你自己安装的cuda版本/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
```
6. 配置生效后，输入nvcc -v，有正确版本输出则代表安装成功

7. cuda是可以安装多个版本的，所以如果有其他人需要其他的版本，管理员可以按照上述步骤安装其他版本的cuda。之后需要使用额外版本的同学，只需要打开自己家目录下的.bashrc，单独配置需要的cuda版本即可。
```bash
# 下面命令由需要额外版本的同学自己配置，相应的cuda需要服务器有安装，没安装的话请先找管理员进行安装

# 打开自己家目录下的.bashrc文件
vim ～/.bashrc

# 在文件最后加入如下两行
# 需要的cuda版本，可以去/usr/local下看具体的名字
export PATH=/usr/local/需要的cuda版本/bin${PATH:+:${PATH}}
export LD_LIBRARY_PATH=/usr/local/需要的cuda版本/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
```

## CUDNN
1. 去官网下载cudnn，注意要和安装的cuda匹配。下载linux_x86_64的tar版本的就行。
2. 安装官网的[手册](https://docs.nvidia.com/deeplearning/cudnn/install-guide/index.html)进行安装，手册导航里选择 installing on linux -> tar file installation。
3. 我写该文时，手册中对应的命令如下：
```bash
# you'll need to replace X.Y and v8.x.x.x with your specific CUDA and cuDNN versions and package date.
# 记得替换下面xx、X.Y等部分
tar -xvf cudnn-linux-x86_64-8.x.x.x_cudaX.Y-archive.tar.xz

sudo cp cudnn-*-archive/include/cudnn*.h /usr/local/cuda/include
sudo cp -P cudnn-*-archive/lib/libcudnn* /usr/local/cuda/lib64
sudo chmod a+r /usr/local/cuda/include/cudnn*.h /usr/local/cuda/lib64/libcudnn*
```

整体来说，cudnn安装比较简单。但是有些注意点，比如上面的安装命令，很明显它是把文件复制到/usr/local/cuda里了。这个文件夹本质是一个软链，指向一个系统上最后安装的那个版本的cuda。所以如果你是专门为某一个版本的cuda安装cudnn，记得把命令后面的/usr/local/cuda目录改成/usr/local目录下实际你需要安装的cuda文件夹。

# 服务器配置
## 使用密钥登陆
应学校网络中心的要求，为了确保服务器的安全，现在一律都需要使用密钥进行登陆。在介绍如何配置密钥登陆前，先简单介绍下密钥。
简单起见，整体用白话进行介绍。用户首先利用自己的电脑中的ssh工具生成一对钥匙。这一对钥匙中有一把是需要交给服务器的，称为公钥。另外一把需要留给自己，不给任何人，称之为私钥。
这两把钥匙有个很神奇的地方就是可以使用私钥解开公钥加密的文件。因此在服务器接收到来自用户的连接请求后，会使用之前保存在自己这里的公钥加密一段随机的字符串，并把这个加密后的字符串发给用户。用户通过机子上的私钥解开加密的字符串并发给服务器。服务器将收到的字符串和原先的字符串进行比较，如果一致，则允许登陆。
通过上面的介绍可以知道，服务器需要实现密钥登陆，就必须让每个用户在自己的电脑上生成这对密钥。接着用户把自己的公钥发给管理员，管理员将公钥放在每个用户家目录下的指定位置，就可以实现密钥登陆了。

### 生成密钥（用户部分）
这里介绍使用终端的方法，如果用户使用图形化界面的ssh连接工具，应该有对应的图形化操作方式，可以自行去搜索引擎上查找教程。
1. 生成的方法也很简单，直接在终端里键入以下命令。有个注意点，就是如果你的机子之前有弄过SSH密钥和某一个Git平台进行关联（如Github、Gitee等）。那么其实你的机子上是有存在密钥文件的，就可以直接使用，不需要再次生成。
```bash
ssh-keygen
```
2. 命令执行过程会出现如下的提示：
```bash
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): <== 它会提供一个默认位置，Linux下是保存在这，windows不一样。注意看，之后文件要从这里找，当然你也可以重新输入指定一个文件夹。
Created directory '/root/.ssh'.
Enter passphrase (empty for no passphrase):＜==输入密钥锁码，直接按 Enter 留空就行
Enter same passphrase again: <== 没有设置密钥锁码的话，直接按 Enter 留空就行
Your identification has been saved in /root/.ssh/id_rsa. <== 私钥保存地址
Your public key has been saved in /root/.ssh/id_rsa.pub. <== 公钥保存地址
```
3. 把生成的公钥发给管理员（到此，用户已经没有需要继续操作的东西了）。

### 给用户配置公钥（管理员部分）
这里的配置方式需要多次切换到用户的账户，其实没必要这么烦琐，但是这样不容易出问题，新的管理员为了避免出错，建议还是按部就班的按照下面的方式进行。
1. 先把服务器用户发给你的公钥先拷贝你自己的账户目录下，我以家目录和scp命令为例（很多人用的是图形化的终端软件，可以直接拖拽复制）。
```bash
scp ~/Desktop/id_rsa.pub 用户名@服务器地址:~/.
```
2. 拷贝公钥到那个用户的家目录下，他的家目录下可能没有.ssh文件夹，建议切到他账户，然后创建。（这里假设这个用户在服务器上的账户名叫A）。
```bash
## 如果用户家目录没有.ssh文件夹，则按照下面步骤进行文件夹创建
su A # 会提示输入A的密码，如果不知道去问下用户，如果他忘记的话，直接用passwd命令来改密码然后再切换
cd ~ # 切换到A账户的家目录
mkdir .ssh
exit # 退回到管理员自己的账户

## 用户家目录下有.ssh命令，或者已经按照上面的方式创建了，则继续按以下步骤进行
sudo cp 公钥文件 /home/A/.ssh

## 更改公钥的所有者为用户A
sudo chown A:A /home/A/.ssh/公钥文件
```
3. 最后的文件配置，包括文件夹权限配置。
```bash
su A # 先切换到A用户，同样如果不知道密码，就去找用户或者passwd命令改密码
cd ~/.ssh # 切换到A用户的家目录下的.ssh文件夹
cat id_rsa.pub >> authorized_keys # 把公钥的内容保存到authorized_keys文件里

# 权限配置
chmod 600 authorized_keys
chmod 700 ~/.ssh

exit # 退回管理员自己的账户
```

### 配置服务器的SSH，禁用密码登陆（管理员部分）
执行完这一步后，服务器就不能用密码登陆了，所以管理员请务必确认在关闭密码前，已经按照前面的生成密钥和配置密钥步骤将自己的账户实现了密钥登陆。（验证方式很简单，重新登一次服务器，不需要输入密码，可以直接登陆即可）

1. 后续给其他用户配置密钥，则不需要再次开启密码登陆，只要管理员可以密钥登陆，就可以按照 **给用户配置公钥（管理员部分）** 的步骤给用户配置密钥。
```bash
sudo vim /etc/ssh/sshd_config # 打开ssh配置文件

# 这两行可能在配置文件里会找不到，这个时候可以在文件的最尾巴，自己加入这两行。（注意大小写，建议直接复制）
RSAAuthentication yes # 打开密钥登陆
PubkeyAuthentication yes # 同上

# 保存退出文件后，执行下面命令
service sshd restart # 重启服务，本来应该同时配置关闭密码登陆的，但是为了安全，先重启一次，实验下能不能密钥登陆
```

2. 在确定管理员自己的账户可以使用密钥登陆后，继续下面的配置，将密码登陆关闭
```bash
sudo vim /etc/ssh/sshd_config # 再次打开配置文件

PasswordAuthentication no # 关闭密码登陆

# 保存退出文件后，执行下面命令
service sshd restart # 重启服务
```

## 服务器IP静态分配
服务器的静态IP分配，根据所使用的ubuntu的系统不同，分为两种方式。在此之前，有个预备知识，即需要会查询本机的网卡名。

### 查本机的网卡名
方法很简单，直接使用ifconfig命令，来获取设备名。当键入ifconfig命令后，会出现诸如如下的信息：
```
enp0s8: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.56.9  netmask 255.255.255.0  broadcast 192.168.56.255
        inet6 fe80::a00:27ff:fe65:4629  prefixlen 64  scopeid 0x20<link>
        ether 08:00:27:65:46:29  txqueuelen 1000  (Ethernet)
        RX packets 9921  bytes 1482806 (1.4 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 10185  bytes 6187094 (6.1 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```
对于如上的信息，enp0s8即为设备名。

### Ubuntu系统版本小于等于16.04
配置文件的地址在/etc/network/interfaces.d文件里，文件的内容大致如下：
```yaml
# 自动分配，（如果是自动分配的，则说明不需要任何配置）
auto 网卡名（比如enp0s8）

# 手动配置（大多数机子是手动配置的）
iface emp0s8 inet static
address xx.xx.xx.xx
netmask xx.xx.xx.xx
gateway xx.xx.xx.xx
dns-nameserver xx.xx.xx.xx
```
如果后续安装的系统版本也一样，则只需要把内容拷贝到新的系统下。
但是如果需要安装新的版本，那么需要自行将这里相应的内容转化为新系统下的格式。

### Ubuntu系统版本大于16.04
配置文件的地址在/etc/netplan文件夹下，这个文件夹下一般会存在一个yaml格式的配置文件，比如01-netcfg.yaml。文件的内容大致如下：
```yaml
# 自动分配
network:
  version: 2
  renderer: networkd


# 手动分配
network:
  version: 2
  renderer: networkd
  ethernets:
    emp0s8:
      addresses: [ip地址/网络掩码(如27)] # 如果是从16.04旧版本系统转到新的版本，那么ip地址对应于address，网络掩码对应于netmask（注意要转化）
      gateway4: 网关地址，对应旧版本的gateway
      nameservers:
        addresses: [dns域名地址]
      dhcp4: no
      optional: no
```


## 服务器硬盘挂载
新买回来的硬盘还需要进行格式化之类的操作，这个很简单，教程很多，就不在这里赘述。（最傻瓜式的方式，就是插到有图形化界面的系统里，然后用系统下的disk工具进行格式化。）

1. 输入下面命令，即可查看磁盘分区的UUID和类型
```bash
sudo blkid

# 输出信息类似如下，需要记住uuid和类型
/dev/sda1: UUID="8048997a-16c9-447b-a209-82e4d380326e" TYPE="ext4"
```

2. 配置系统文件，配置文件的地址在/etc/fstab
```bash
vim /etc/fstab

# 配置如下
UUID=8048997a-16c9-447b-a209-82e4d380326e /media/data1 ext4 defaults 0 1

上面配置的每列代表了UUID、挂载点、硬盘文件类型，后面三列默认 defaults,0,1即可
```

## 防火墙设置
防火墙设置是安装系统后都需要做的，在设置防火墙之前，需要知道实验室的对外唯一IP地址（我们实验室内部都用的局域网）。然后进行设置，设置的命令也比较简单。
```bash
sudo ufw allow from xx.xx.xx.xx(IP地址) to any port 22
sudo ufw enable
```

# 常见问题

## 明明在用户家目录的.ssh文件夹里配置了公钥文件，却不能登陆。
首先用ls -al命令检查下该用户家目录的.ssh文件夹里全部文件的所有者，一般来说，这些文件的所有者都应该是这个用户。如果不是的话，用chown命令给他改一下。之后再检查下authorized_keys文件和.ssh文件夹的权限，它们的权限要分别是600，700（600，700如果不知道什么意思的话，看这个[博客](https://www.cnblogs.com/mingbai/p/linuxPermission2Num.html)）。
还不行的话，可能是公钥文件的问题。公钥文件的内容多一个空格都不行，建议重新从用户电脑上传一个公钥，然后重新按照步骤配置下。**之前就有发生过因为偷懒直接使用文本复制，结果不能用的情况。**

## 能不能让用户自己传公钥
答案是可以，但是用户自己搞容易出各种各样的问题，还是建议管理员统一帮他们配置。

## 服务器重装后，用户无法操控机械硬盘里原数据文件
出现这个问题的原因是，在系统重装后，原先的用户都被清空了。所以新建的用户对于它之前的文件夹没有控制权限。所以只需要把文件的所有者改成新创建的用户就行。