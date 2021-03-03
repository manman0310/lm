---
layout: blog_default
title:  "自动化构建ios APP"
---

# 自动化构建ios APP

**折腾一下ios APP的Jenkins自动构建，下一步准备自动提交到苹果商店**
**APP采用cordova+ionic框架，CICD工具继续拥抱Jenkins**

环境：
 >  Jenkins slave OS： Mac OS X 10.15.7
 >  Nodejs：12.10.0  
 >  Cordova：9.0  
 >  jdk:：1.8  
 >  yarn：1.19.1  

***

## 过程

### 配置Jenkins在mac上的slave

首先slave主机需要jdk

<img src="../../../assets/image/noJdkError.png" alt="未安装jdk错误" width="700" />

mac设置一下固定ip，简单配置一下，launch。

<img src="../../../assets/image/JenkinsConfig.png" alt="Jenkins配置" width="800" />

左下角这样就是连上了slave

<img src="../../../assets/image/jenkinsSlaveSuccess.png" alt="Jenkins配置成功" width="400" />

***

## 踩坑及爬坑

### 0. 因为程序编译过程需要在docker容器里进行，所以需要在slave上登录公司docker仓库。如此简单的一步也不顺利

**报错信息：**  

``` shell
Error saving credentials: error storing credentials - err: exit status 1, out: `Cannot autolaunch D-Bus without X11 $DISPLAY`
```

**解决方案：**  

``` shell
root@k8s-master1:~# apt purge golang-docker-credential-helpers
```

### 1. 在jenkins 中执行shell脚本，并不能读取到环境变量，导致一堆命令找不到

**报错信息：**  
jenkins执行shell读不到环境变量  

**报错原因：**  
https://blog.csdn.net/zzusimon/article/details/57080337?depth_1-
这个里面讲的很详细，感谢大神指点。  
说白了就是shell读取不到/etc/profile以及~/.bash_profile里面的环境变量。  

**解决方案：**  
shell第一行

``` shell
#!/bin/bash -ilex
```

-e : 出错则退出shell  
-x ：显示执行的每一行命令 
-i : 交互式脚本  
-l : 登录式脚本  

### 2. xcode 编译出现 errSecInternalComponent error，这个问题折腾良久  

**报错信息：**  
xcode 编译出现:  

``` shell
errSecInternalComponent error  
```  

**报错原因：**  
可能是xcode签名机制(code signing mechanism) 的 bug, Xcode 中账号多了，就会产生很多过期的PP文件，Xcode 没有自带删除功能会导致重复导入provisioning profile.  
可能是在mac上直接界面操作会弹出登录钥匙串的输入密码框，用shell执行则就会无法登录。  
可能是项目的缓存问题。  

**尝试解决一：**  
Xcode 中所有的PP文件，都在 ~/Library/MobileDevice/Provisioning Profiles 这个文件夹下；进入该文件夹，删除不需要的；重新导入新的 provisioning profile  
然后重启Mac；  

**尝试解决二：**  

``` shell
security unlock-keychain -p "123456" ~/Library/Keychains/login.keychain-db  
```  
执行前解锁钥匙串，看上去可以解决但是实际没效果    

**尝试解决三：**  
项目根目录执行xattr -rc . 用来删除文件的扩展属性  

**尝试解决四：**  
移除~/Library/Developer/Xcode/DerivedData/目录下所有文件  

**最终解决：**  
``` shell
security set-key-partition-list -S apple-tool:, apple: -s -k "123456" ~/Library/Keychains/login.keychain-db  
```  
目的是赋予apple-tool:, apple两个服务系统的钥匙串权限。  
p.s. 老版本macOS默认存在 login.keychain 中，而升级到10.12后会存在 login.keychain-db中  
  

### 3. 自动build ipa会提示描述文件中没有推送功能  

**报错信息：**  
``` shell
‘***APP’ requires a provisioning profile with the Push Notifications feature. Select a provisioning profile in the Signing & Capabilities editor.   
```  

**报错原因：**  
当去比较手动Archive打出ipa时，发现 ExportOptions.plist内容和原来的plist，相差几组数据。  
Xcode升级之后，多了这个ExportOptions.plist文件。   
<img src="../../../assets/image/noPlistError.png" alt="没有plist文件" width="400" />  

**解决方案：**  
脚本中执行cordova build之前，增加拷贝该文件到ios项目根目录操作。  

### 4. 极光推送配置没有跟随目标ipa环境修改IsProduction属性值  

**报错信息：**  
执行cordova build ios --release之后，项目JPushConfig.plist中IsProduction依然是false。导致在生产环境极光推送无法使用。     
<img src="../../../assets/image/jpushConfigError.png" alt="极光配置错误" width="600" />  

**报错原因：**  
需要手工修改。     

**解决方案：**  
在执行cordova build之前执行cordova hook（before_build），通过是debug还是release修改值为false或者true。  
虽然，cordova官网推荐使用js脚本，但是由于需要修改plist文件，nodejs改这个类型的文件比较麻烦，所以还是决定使用sh脚本。    

### 5. 执行build ios前的hook shell脚本时报错   

**报错信息：**  

``` shell
hook Command finished with error code EACCES   
```

**报错原因：**  
因为是shell脚本，所以存在权限问题。      

**解决方案：**  
在执行cordova build之前赋予该sh文件权限。  

``` shell
chmod 777 shell    
```   
