# Linux学习(以升级Ubuntu系统为例，从20升级到22)

## 购买配置阿里云服务器
配置
Ubuntu 22.04 64位
2 核（vCPU）
2 GiB

## 对Ubuntu22升级

### 看目前系统状态

```
lsb_release -a // 确认系统版本

LSB Version:    core-11.1.0ubuntu4-noarch:security-11.1.0ubuntu4-noarch
Distributor ID: Ubuntu
Description:    Ubuntu 22.04.5 LTS
Release:        22.04
Codename:       jammy

uname -r // 查看当前内核

5.15.0-186-generic

df -h / // 查看root分区容量

Filesystem      Size  Used Avail Use% Mounted on
/dev/vda3        40G  6.5G   31G  18% /

```

### 查看并备份软件源

```
cat /etc/apt/sources.list   // 查看阿里云配值的什么源

## Note, this file is written by cloud-init on first boot of an instance
## modifications made here will not survive a re-bundle.
## if you wish to make changes you can:
## a.) add 'apt_preserve_sources_list: true' to /etc/cloud/cloud.cfg
##     or do the same in user-data
## b.) add sources in /etc/apt/sources.list.d
## c.) make changes to template file /etc/cloud/templates/sources.list.tmpl

# See http://help.ubuntu.com/community/UpgradeNotes for how to upgrade to
# newer versions of the distribution.
deb http://mirrors.cloud.aliyuncs.com/ubuntu jammy main restricted
# deb-src http://mirrors.cloud.aliyuncs.com/ubuntu jammy main restricted

## Major bug fix updates produced after the final release of the
## distribution.
deb http://mirrors.cloud.aliyuncs.com/ubuntu jammy-updates main restricted
# deb-src http://mirrors.cloud.aliyuncs.com/ubuntu jammy-updates main restricted

## N.B. software from this repository is ENTIRELY UNSUPPORTED by the Ubuntu
## team. Also, please note that software in universe WILL NOT receive any
## review or updates from the Ubuntu security team.
deb http://mirrors.cloud.aliyuncs.com/ubuntu jammy universe
# deb-src http://mirrors.cloud.aliyuncs.com/ubuntu jammy universe
deb http://mirrors.cloud.aliyuncs.com/ubuntu jammy-updates universe
# deb-src http://mirrors.cloud.aliyuncs.com/ubuntu jammy-updates universe

## N.B. software from this repository is ENTIRELY UNSUPPORTED by the Ubuntu
## team, and may not be under a free licence. Please satisfy yourself as to
## your rights to use the software. Also, please note that software in
## multiverse WILL NOT receive any review or updates from the Ubuntu
## security team.
deb http://mirrors.cloud.aliyuncs.com/ubuntu jammy multiverse
# deb-src http://mirrors.cloud.aliyuncs.com/ubuntu jammy multiverse
deb http://mirrors.cloud.aliyuncs.com/ubuntu jammy-updates multiverse
# deb-src http://mirrors.cloud.aliyuncs.com/ubuntu jammy-updates multiverse

## N.B. software from this repository may not have been tested as
## extensively as that contained in the main release, although it includes
## newer versions of some applications which may provide useful features.
## Also, please note that software in backports WILL NOT receive any review
## or updates from the Ubuntu security team.
deb http://mirrors.cloud.aliyuncs.com/ubuntu jammy-backports main restricted universe multiverse
# deb-src http://mirrors.cloud.aliyuncs.com/ubuntu jammy-backports main restricted universe multiverse

deb http://mirrors.cloud.aliyuncs.com/ubuntu jammy-security main restricted
# deb-src http://mirrors.cloud.aliyuncs.com/ubuntu jammy-security main restricted
deb http://mirrors.cloud.aliyuncs.com/ubuntu jammy-security universe
# deb-src http://mirrors.cloud.aliyuncs.com/ubuntu jammy-security universe
deb http://mirrors.cloud.aliyuncs.com/ubuntu jammy-security multiverse
# deb-src http://mirrors.cloud.aliyuncs.com/ubuntu jammy-security multiverse

// 类似mirrors的地址是阿里云预置的镜像源

sudo cp /etc/apt/source.list /etc/apt/source.list.bak   // 做好备份

```

### 把20.04更新到最新状态

```
// 养成好习惯，做任何处理前先升级包
sudo apt update // 刷新软件列表
sudo apt upgrade -y // 升级已经安装的包
sudo apt autoremove -y // 自动清理掉没用的依赖包
```

### 安装升级工具，并开一个防断线保险

```
sudo apt install update-manager-core tmux -y    // 先安装tmux
tmux    // 启动tmux服务
// tmux是一个终端复用器，因为全程升级需要20-40min，如果ssh意外断开，升级进程会直接卡死，系统会损坏，在tmux里跑，断线重连后执行tmux attach就能返回
```

### 正式升级

```
sudo do-release-upgrade     // 开始升级前会做安全检查，检测到有未安装的更新便退出
Checking for a new Ubuntu release
Please install all available updates for your release before upgrading.

// 处理问题
sudo apt dist-upgrade -y    // 智能地增删依赖，把这类包也升级掉。

ls /var/run/reboot-required     // 检查是否需要重启,提示No such file or directory，不需要重启
sudo reboot     // 重启

// 问题没解决
sudo apt upgrade    // 检查哪一个包没有更新成功
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
Calculating upgrade... Done
Get more security updates through Ubuntu Pro with 'esm-apps' enabled:
  gsasl-common libgsasl7 python3-pip
Learn more about Ubuntu Pro at https://ubuntu.com/pro
The following packages have been kept back:
  cloud-init
0 upgraded, 0 newly installed, 0 to remove and 1 not upgraded.
// 发现cloud-init被保留，通常有两种原因：1、新版要引入新依赖，upgrade 不处理这种情况；2、Ubuntu 的「分阶段更新」（phased updates）机制把你这台机器排在了后面的批次。
// 显示指定包名更新
sudo apt install cloud-init -y
// 需要把一些包解除锁定，先看一下哪些被锁了
sudo apt-mark showhold
cloud-init
intel-microcode
// 解锁
sudo apt-mark unhold cloud-init intel-microcode
// 然后手动升级
sudo apt upgrade -y
// 查找一下还有没有别的包被锁定了，再看看需不需要重启
apt-mark showhold
ls /var/run/reboot-requierd

// 没有，问题解决，开始升级
sudo do-release-upgrade

== More Information ==

You can find out more about Ubuntu on our website, IRC channel and wiki.
If you're new to Ubuntu, please visit:

  http://www.ubuntu.com/


To sign up for future Ubuntu announcements, please subscribe to Ubuntu's
very low volume announcement list at:

  http://lists.ubuntu.com/mailman/listinfo/ubuntu-announce

Continue [yN]y

Checking for installed snaps

Calculating snap size requirements

Updating repository information

No valid mirror found

While scanning your repository information no mirror entry for the
upgrade was found. This can happen if you run an internal mirror or
if the mirror information is out of date.

Do you want to rewrite your 'sources.list' file anyway? If you choose
'Yes' here it will update all 'jammy' to 'noble' entries.
If you select 'No' the upgrade will cancel.

Continue [yN] n     // 发现官方想把我直接推送到24.04版本，取消升级，noble是24.04的代号，jammy是22.04的代号

// 回退，查看版本是否是22.04
lsb-release -a
LSB Version:    core-11.1.0ubuntu4-noarch:security-11.1.0ubuntu4-noarch
Distributor ID: Ubuntu
Description:    Ubuntu 22.04.5 LTS
Release:        22.04
Codename:       jammy
// 发现版本就是22，查看服务器控制台，发现也是22版本，说明在最开始就搞错了，现在更改目标，手动从22升级到24
```

### 切换软件源
```
cat /etc/apt/sources.list                                  // 先看清内容
sudo cp /etc/apt/sources.list /etc/apt/sources.list.jammy.bak   // 备份
sudo sed -i 's/jammy/noble/g' /etc/apt/sources.list        // 核心操作：全量替换代号
cat /etc/apt/sources.list                                  // 检查替换结果
```

### 拉取新版本软件索引
```
sudo apt upgrade        // 从阿里云源拉取数据

sudo apt upgrade --without-new-pkgs -y     // 不引入新包，先把已有的包换到noble版本

sudo apt full-upgrade -y        // 允许增删包，彻底的改版

// 完成后，执行
sudo apt autoremove -y      // 删掉一些没用的包
sudo reboot     // 重启

// 重新连接后验证看看
lsb_release -a
No LSB modules are available.
Distributor ID: Ubuntu
Description:    Ubuntu 24.04.4 LTS
Release:        24.04
Codename:       noble

uname -r
5.15.0-186-generic


sudo dpkg -C        // 审计包状态：无输出 = 没有一个包处于损坏/半配置状态
```