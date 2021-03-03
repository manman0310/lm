---
layout: blog_default
title:  "常用命令备忘录"
---

# 常用命令备忘录

省的再百度谷歌到处查，持续更新中

### git

``` sh 
git reset --hard origin/master  // 强制reset到某个版本不保留本地更改

```

### 开发常用

``` sh
ionic cordova resources  // 生成移动端资源包
increase-memory-limit  // 跑npm超内存限制下这个包可以解决

```

### 运维常用

``` sh
tracert  // 路由跟踪
free -lh  // 查看内存
/sbin/iptables -I INPUT -p tcp --dport 9999 -j ACCEPT  // linux防火墙放开端口
cat /proc/cpuinfo  // 查看 cpu信息
grep MemTotal /proc/meminfo  // 查看内存信息
uname -a  // 查看操作系统信息
cat /etc/issue  // 查看centos版本信息
df -h  // 查看磁盘使用情况
fdisk -l  // 查看其它磁盘外设信息
lsblk  // 查看所有可用块设备的信息
dmidecode |more  // 查看所有硬件信息
dmesg |more  // 或：查看所有硬件信息
ethtool eth0  // 查看网卡信息
```

### docker

``` sh 
docker sysrtem df  // 重启 docker 服务
sudo systemctl restart docker  // docker镜像容器占用空间查看
```
