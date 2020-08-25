---
layout: blog_default
title:  "常用命令备忘录"
---

# 常用命令备忘录

省的再百度谷歌到处查，持续更新中

### git
```
git reset --hard origin/master  // 强制reset到某个版本不保留本地更改

```

### 开发常用
```
ionic cordova resources  // 生成移动端资源包
increase-memory-limit  // 跑npm超内存限制下这个包可以解决

```

### 运维常用
```
tracert  // 路由跟踪
free -lh  // linux查看内存
/sbin/iptables -I INPUT -p tcp --dport 9999 -j ACCEPT  // linux防火墙放开端口

```

### docker
```
docker sysrtem df  // 重启 docker 服务
sudo systemctl restart docker  // docker镜像容器占用空间查看
```


