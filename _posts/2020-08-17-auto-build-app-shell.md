---
layout: blog_default
title:  "基于cordova持续集成app脚本"
tag: [CICD, cordova, android, shell]
---

# 基于cordova持续集成app脚本

## 场景

**公司需要项目定制化APP，其实就是基于公司产品APP（Android）进行的二次定制，替换应用信息以及资源之后自动打包即可。**

## 环境准备

**cordova 9.0的基础镜像，具体如下，镜像dockerfile以后会贴出来**

01. nodejs 12.10.0 
02. yarn 1.22.4
03. cordova 9.0
04. xmlstarlet插件 用来修改xml文件
05. android SDK 28
06. gradle 4.10.3

## 脚本说明

### 生成安卓签名脚本：【generate-keystore.sh】

<a class="u-post-a" href="../../../assets/resource/generate-keystore.html" target="_blank">点 · 这 · 里 · 查 · 看 · 脚 · 本 ~</a>  

##### 传入参数

|序号|参数名|对应变量|值示例|是否必传|注释|
|:-:|:-:|:-:|:-:|:-:|-|
|1|-a|APP_KEYWORD|delian|是|-|

##### 默认参数

|变量名|示例|注释|
|:-:|:-:|-|
|GLOBAL_NEXUX|https://nexus.flexem.net|readonly|
|GLOBAL_NEXUX_BASICAUTH|wwb:1qaz! QAZ|readonly|

##### 返回状态码

|状态码|意义|
|:-:|:-:|
|0|成功|
|1|未传入任何参数|
|2|-a参数未传值|
|3|无效入参|
|4|生成签名失败|
|5|上传仓库失败|

##### 示例

``` bash
./generate-keystore.sh -a delian
```

***

### 构建预备脚本：【prepare.sh】

<a class="u-post-a" href="../../../assets/resource/prepare.html" target="_blank">点 · 这 · 里 · 查 · 看 · 脚 · 本 ~</a> 

##### 传入参数

|序号|参数名|对应变量|是否有值|值示例|是否必传|备注|
|:-:|:-:|:-:|:-:|:-:|:-:|-|
|1|-p|后接入参|否|无值|是|
|2|--docker-tag|DOCKER_IMAGE_TAG|是|"4.0"|是|-|
|3|--docker-img|DOCKER_IMAGE_URL|是|docker.flexem.com/flexem-fcloud/cordova9-tool|是|-|
|4|--app-key|APP_KEYWORD|是|delian|是|-|
|5|--app-name|APP_NAME|是|德联|是|如果有空格，请用英文单引号括起内容|
|6|--site-url|SITE_URL|是|delian-yun.cn|是|-|
|7|--gaode-key|GAODE_KEY|是|-|是|-|
|8|--jpush-key|JPUSH_KEY|是|-|否|-|
|9|--git-branch|GIT_BRANCH|是|delian-proj|否|default using master branch|
|10|--git-tag|GIT_TAG_NAME|是|2.1.0|否|【内部字段】default using latest version|
|11|--is-mulit|IS_MULTI_TENANT|否|无|否||
|12|--is-https|IS_HTTPS|否|无|否|-|
|13|--app-color|APP_COLOR|是|blue|否|【预留字段】default using blue skin|

##### 默认参数

|变量名|示例|注释|
|:-:|:-:|-|
|DOCKER_IMAGE_NAME|cordova9-tool|readonly|

##### 返回状态码

|状态码|意义|
|:-:|:-:|
|0|成功|
|101|未传入任何参数|
|102|缺少必要参数|
|199|无效入参|
|201|镜像拉取失败|
|202|容器运行失败|
|203|拷贝脚本进入容器失败|
|204|容器中运行脚本失败|

##### 示例

``` bash
./prepare.sh -p --docker-tag 4.0 --docker-img docker.flexem.com/flexem-fcloud/cordova9-tool --app-key delian --app-name '德联' --site-url delian-yun.cn --jpush-key ceshijpushkey123456 --gaode-key ceshigaodekey123456 --git-branch delian-branch --git-tag delian-tag --is-mulit --is-https --app-color blue
```

***

### 构建过程脚本：【build.sh】  

<a class="u-post-a"  href="../../../assets/resource/build.html" target="_blank">点 · 这 · 里 · 查 · 看 · 脚 · 本 ~</a>  

##### 传入参数 & 默认参数

无需操作

##### 返回状态码

|状态码|意义|
|:-:|:-:|
|0|成功|
|101|资源拉取失败|
|102|git克隆失败|
|103|yarn执行失败|
|104|npm run build执行失败|
|105|cordova生成resources执行失败|
|106|add安卓平台失败|
|107|build安卓平台失败|
|201|apk签名失败|
|202|apk上传失败|

***
  

## 【steps】 步骤

01. pull docker image and run
02. copy build.sh into container and run shell
03. clone from git, get resources, get keystore
04. update infos in config.xml
    - change APK key =====> flexem.$APP_KEYWORD.assistant
    - change name & description =====> $APP_NAME
    - change jpush key =====>  $JPUSH_KEY

05. update infos in agconnect-services.json
    - change APK key =====> flexem.$APP_KEYWORD.assistant

06. update infos in config.json
    - change IsMultiTenants =====> $IS_MULTI_TENANT
    - change DefaultDomainName =====> $SITE_URL

07. update infos in package.json
    - change site url  =====> $SITE_URL
    - change enableHttps =====> $IS_HTTPS
    - change jpush key =====>  $JPUSH_KEY

08. update infos in src/index.html
    - change gaode key =====> $GAODE_KEY

09. yarn
10. update domain-logo-default.gif
    - if domain.png exists than copy into src/assets/img and named domain-logo-default.gif

11. npm run build
12. copy new resources into mobile/resources
    - copy icon.png
    - copy splash.png

13. ionic cordova resources
14. cordova platform add android
15. cordova build android
16. jarsigner to sign apk
17. upload $APP_KEYWORD.apk to $APP_KEYWORD file
