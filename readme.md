# Linux学习(以升级Ubuntu系统为例，从22升级到24)

## 购买配置阿里云服务器
配置
Ubuntu 22.04 64位
2 核（vCPU）
2 GiB

## 对Ubuntu24升级

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
cat /etc/apt/sources.list   // 查看阿里云配置的是什么源

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

sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak   // 做好备份

```

### 把22.04更新到最新状态

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
ls /var/run/reboot-required

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
lsb_release -a
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
sudo apt update        // 从阿里云源拉取数据

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

### 内核没有升级成功（裸内核问题）

```
// 升级完成后发现内核还是旧版本
uname -r
5.15.0-186-generic

ls /boot/vmlinuz-*
/boot/vmlinuz-5.15.0-186-generic    // 只有旧内核，6.8 根本没装上

// 查看内核包家族
dpkg -l | grep -E 'linux-(image|headers|generic|virtual|modules)'
// 结果只有 linux-image-5.15.0-186-generic 等裸内核包，
// 没有 linux-generic 之类的元包

// 诊断结论：云镜像装的是"裸内核"，没有元包
// 元包（metapackage）本身不含内核文件，只是"指向最新内核"的空壳
// apt 眼里系统是完整的：旧内核没有更新可用，新内核没人点名要装
// 所以 full-upgrade 不会自动装 6.8 新内核

sudo apt install linux-generic -y    // 药方：手动补装元包，自动拉齐 6.8 内核+模块+头文件
ls /boot/vmlinuz-*                   // 现在能看到 5.15 和 6.8 两个内核
sudo reboot                          // 内核切换必须重启——运行中的内核是开机那一刻加载进内存的

uname -r
6.8.0-XX-generic                     // 重启后新内核生效 ✔
```

### 升级过程中的坑（复盘记录）

1. **grub-pc 对话框**：升级引导程序 GRUB 时弹出全屏菜单（键盘操作：方向键移动、空格勾选、Tab 切到 Ok、回车）。必须选**整块磁盘 /dev/vda**，不能选分区 /dev/vda3——分区引导用 blocklist 机制，不可靠。
2. **needrestart 与 aegis 崩溃**：装完包后 needrestart 检查哪些服务还在用旧库并自动重启。其中 `aegis.service` 启动失败——aegis 是阿里云云盾的安全代理（第三方商业软件），跨版本升级后旧二进制与新系统库不兼容而崩溃。**不影响系统本身**，遗留待处理。
3. **幽灵文件**：/etc/apt/ 下出现 `sources.list.curtin.orig`（云镜像安装工具 curtin 留的出厂原文件）和 `sources.list.distUpgrade`（被中止的 do-release-upgrade 留的备份）。apt 只读 `sources.list` 和 `sources.list.d/` 下 `.list` 结尾的文件，其他后缀一律无视。**千万别把它们改名成 .list 结尾**，否则旧源会被重新读进来。
4. **"版本号新 ≠ 升级完成"**：lsb_release 显示 24.04 时内核可能还是旧的。`apt list --upgradable` 有盲区——只列"已装包有新版本"，看不见"该新装还没装"的包（新内核属于后者：5.15 和 6.8 是不同包名）。
5. **do-release-upgrade 不认阿里云镜像**：官方工具识别不了 mirrors.cloud.aliyuncs.com 的源格式，提示 "No valid mirror found"。手动方案（sed 改源）完美绕开，且全程走阿里云镜像，下载快。

### 本次学到的核心概念

| 概念 | 一句话解释 |
|---|---|
| apt update / upgrade / full-upgrade / autoremove | 刷新软件列表 / 升级已装包 / 允许增删包的彻底升级 / 清理无用包 |
| apt-mark hold / unhold | 锁定 / 解锁包版本，防止被升级 |
| 软件源 sources.list | apt 的"货架清单"；换版本 = 换代号（jammy → noble） |
| 元包 metapackage | 不含实际文件的"指向最新版本"的空壳，如 linux-generic |
| 内核切换必须重启 | 运行中的内核是开机那一刻加载进内存的 |
| 快照 | 变更前的兜底，失败可一键回滚 |
| tmux | 终端复用器，SSH 断线不中断正在跑的任务 |
| needrestart | 升级后检查哪些服务需要重启的小管家 |
| GRUB | 引导程序，开机时负责加载内核 |
| 第三方软件兼容性 | 云盾 aegis 这类贴底层的软件，系统大版本升级时最容易崩 |
