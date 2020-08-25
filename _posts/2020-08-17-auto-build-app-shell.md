---
layout: blog_default
title:  "基于cordova自动打包app脚本"
tag: 脚本集合
---

# 基于cordova自动打包app脚本

## 场景

**公司需要项目定制化APP，其实就是基于公司产品APP进行的二次定制，替换应用信息以及资源之后自动打包即可。**

## 环境准备

**cordova 9.0的基础镜像，具体如下，镜像dockerfile以后会贴出来**

01. nodejs 12.10.0 
02. yarn 1.22.4
03. cordova 9.0
04. xmlstarlet插件 用来修改xml文件
05. android SDK 28
06. gradle 4.10.3

## 脚本及过程

**脚本分为两个部分**

01. prepare.sh: 准备脚本，主要用来拉镜像起容器，传参数
02. build.sh 构建脚本，在容器中运行并构建

### 【脚本入参（只是我们的实例，可以根据实际情况调整）】

#### prepare.sh

|接收序号|变量名|示例|注释|
|:-:|:-:|:-:|-|
|$1|DOCKER_IMAGE_TAG|4.0|-|
|$2|DOCKER_IMAGE_URL|你的镜像仓地址|-|
|$3|APP_KEYWORD|companyName|-|
|$4|APP_NAME|公司名|-|
|$5|JPUSH_KEY|-|极光key|
|$6|GIT_BRANCH|company-proj|default using master branch|
|$7|GIT_TAG_NAME|2.1.0|【内部字段】default using latest version|
|$8|GAODE_KEY|-|高德地图key|
|$9|SITE_URL|company-yun.cn|-|
|$10|IS_MULTI_TENANT|false|【预留字段】是否多租户|
|$11|IS_HTTPS|false|是否取https|
|$12|APP_COLOR|red|【预留字段】default using blue skin|

#### build.sh （参数值通过第一个脚本直接传进来）

|接收序号|变量名|示例|注释|
|:-:|:-:|:-:|-|
|$1|APP_KEYWORD|delian|-|
|$2|APP_NAME|companyName|-|
|$3|JPUSH_KEY|-|极光key|
|$4|GIT_BRANCH|company-proj|if input is none, it will pull use the master branch|
|$5|GIT_TAG_NAME|2.1.0|if input is none, it will use the latest version|
|$6|GAODE_KEY|-|高德地图key|
|$7|SITE_URL|company-yun.cn|-|
|$8|IS_MULTI_TENANT|false|【预留字段】是否多租户|
|$9|IS_HTTPS|false|是否取https|
|$10|APP_COLOR|red|【预留字段】default using blue skin|

#### default params

|变量名|示例|注释|
|:-:|:-:|-|
|GLOBAL_NEXUX|你的素材仓库地址|-|
|GLOBAL_NEXUX_BASICAUTH|仓库鉴权信息|-|
|DOCKER_IMAGE_NAME|镜像名|-|
|GIT_AUTH|git鉴权信息|-|

### 【执行步骤】

01. pull docker image and run (获取镜像运行容器)
02. copy build.sh into container and run shell (拷贝脚本进去镜像)
03. clone from git, get resources, get keystore (我们会把私有化的素材放在公司仓库，包括icon和启动页还有签名证书)
04. update infos in config.xml (这里是替换app基本信息和极光的key)
    - change APK key =====> 你希望的包名
    - change name & description =====> $APP_NAME
    - change jpush key =====>  $JPUSH_KEY

05. update infos in package.json (这里是替换app基本信息和极光的key)
    - change site url  =====> $SITE_URL
    - change enableHttps =====> $IS_HTTPS
    - change jpush key =====>  $JPUSH_KEY

06. update infos in src/index.html (这里替换高德key)
    - change gaode key =====> $GAODE_KEY

07. yarn
08. npm run build
09. copy new resources into mobile/resources (将新的icon和splash拷贝进项目准备生成素材)
10. ionic cordova resources (通过ionic cordova生成新的素材集合)
11. cordova platform add android (添加安卓平台)
12. cordova build android (编译apk)
13. jarsigner to sign apk (apk签名)
14. upload $APP_NAME.apk to $APP_NAME file (最后再次推仓)

### 【脚本内容】

#### prepare.sh

``` sh
#! /bin/bash

DOCKER_IMAGE_TAG=$1
DOCKER_IMAGE_URL=$2
APP_KEYWORD=$3
APP_NAME=$4
JPUSH_KEY=$5
GIT_BRANCH=$6
GIT_TAG_NAME=$7
GAODE_KEY=$8
SITE_URL=$9
IS_MULTI_TENANT=${10}
IS_HTTPS=${11}
APP_COLOR=${12}

DOCKER_IMAGE_NAME=你的镜像名称

echo "pull docker image :  $DOCKER_IMAGE_URL:$DOCKER_IMAGE_NAME"
docker pull $DOCKER_IMAGE_URL:$DOCKER_IMAGE_TAG

echo "stop docker container :  $DOCKER_IMAGE_NAME"
docker stop $DOCKER_IMAGE_NAME
docker rm $DOCKER_IMAGE_NAME

echo "run docker image"
docker run -itd --name=$DOCKER_IMAGE_NAME --env "CORDOVA_ANDROID_GRADLE_DISTRIBUTION_URL=https://nexus.flexem.net/repository/tools/gradle/gradle-4.10.3-all.zip" --env "SASS_BINARY_SITE=https://npm.taobao.org/mirrors/node-sass/" -u root $DOCKER_IMAGE_URL:$DOCKER_IMAGE_TAG

echo "copy build.sh into docker container"
docker cp ./build.sh $DOCKER_IMAGE_NAME:/opt

echo "into docker container with run shell"
docker exec -it -w /opt $DOCKER_IMAGE_NAME /bin/bash ./build.sh $APP_KEYWORD $APP_NAME $JPUSH_KEY $GIT_BRANCH $GIT_TAG_NAME

```

#### build.sh

``` sh
#! /bin/bash

APP_KEYWORD=$1
APP_NAME=$2
JPUSH_KEY=$3
GIT_BRANCH=$4
GIT_TAG_NAME=$5
GAODE_KEY=$6
SITE_URL=$7
IS_MULTI_TENANT=$8
IS_HTTPS=$9
APP_COLOR=${10}

GIT_AUTH=git鉴权信息
GLOBAL_NEXUX="仓库地址"
GLOBAL_NEXUX_BASICAUTH="仓库鉴权信息"

echo "create /apk"
if [[ ! -d "/apk" ]]; then
    mkdir /apk
else
    rm -rf /apk/*
fi

echo "start update infos in config.xml"
echo "clone from git & get resources & get keystore..."
cd /apk
wget https://nexus.flexem.net/repository/artifact/fcloud-artifact/project/$APP_KEYWORD/icon.png
wget https://nexus.flexem.net/repository/artifact/fcloud-artifact/project/$APP_KEYWORD/splash.png
wget https://nexus.flexem.net/repository/artifact/fcloud-artifact/project/$APP_KEYWORD/$APP_KEYWORD.keystore
chmod 777 $APP_KEYWORD.keystore
git clone git项目仓库地址
cd 项目路径
# git checkout $GIT_TAG_NAME
git checkout $GIT_BRANCH

echo "flexem.fcloud.assistant =====> 你的包名..."
cd mobile
sed -i "s/原产品包名/你的包名/g" config.xml

echo "name & description =====> $APP_NAME..."
xmlstarlet ed -u '/widget/name' -v $APP_NAME config.xml
xmlstarlet ed -u '/widget/description' -v $APP_NAME config.xml

echo "jpush APP_KEY =====>  $JPUSH_KEY..."
xmlstarlet ed -u '/widget/plugin[name="jpush-phonegap-plugin"]/variable[name="APP_KEY"]' -v $JPUSH_KEY config.xml
echo "end update infos in config.xml"

echo "start update infos in package.json"
echo "yunzutai.com =====> $SITE_URL..."
sed -i "s/yunzutai.com/$SITE_URL/g" package.json

echo "enableHttps =====> $IS_HTTPS..."
enableHttps_line=$(awk -F"\"" '/enableHttps/{print NR}' package.json) # 记住行号
enableHttps_old=$(awk '/enableHttps/{print $2}' package.json)  # 获取旧数据
sed -e "$enableHttps_line s@$enableHttps_old@$IS_HTTPS,@" -i package.json # 替换所在行的老数据
sed -e "$enableHttps_line s@$enableHttps_old@$IS_HTTPS,@" -i

echo "jpush APP_KEY =====> $JPUSH_KEY..."
APP_KEY_line=$(awk -F"\"" '/APP_KEY/{print NR}' package.json) # 记住行号
APP_KEY_old=$(awk -F"\"" '/APP_KEY/{print $4}' package.json)  # 获取旧数据
sed -e "$APP_KEY_line s@$APP_KEY_old@$JPUSH_KEY@" -i package.json # 替换所在行的老数据
echo "end update infos in package.json"

echo "start update infos in index.html"
echo "script gaode key =====> $GAODE_KEY..."
cd src
GAODE_KEY_line=$(awk -F"\"" '/AMap/{print NR}' index.html) # 记住行号
GAODE_KEY_old= `cat index.html | grep AMap |awk -F"&" '{print $2}'` # 获取旧数据
sed -e "$GAODE_KEY_line s@$GAODE_KEY_old@key=$GAODE_KEY@" -i index.html # 替换所在行的老数据
echo "end update infos in index.html"

echo "yarn"
cd ../
yarn

echo "npm run build"
npm run build

echo "copy new resources into mobile/resources..."
cp -rf /apk/icon.png resources/icon.png
cp -rf /apk/splash.png resources/splash.png

echo "ionic cordova resources"
ionic cordova resources

echo "cordova platform add android"
cordova platform add android

echo "cordova build android"
cordova build android --release

echo "jarsigner to sign apk"
cd platforms/android/app/build/outputs/apk/release
jarsigner -verbose -keystore /apk/$APP_KEYWORD.keystore -keypass 123456 -storepass 123456 -signedjar $APP_KEYWORD.apk app-release-unsigned.apk $APP_KEYWORD

echo "upload $APP_KEYWORD.apk to $APP_KEYWORD file"
curl -X POST "$GLOBAL_NEXUX/service/rest/v1/components?repository=artifact" -H "accept:application/json" -H "Content-Type:multipart/form-data" -F "raw.directory=fcloud-artifact/project/$APP_KEYWORD" -F "raw.asset1=@$APP_KEYWORD.apk" -F "raw.asset1.filename=$APP_KEYWORD.apk" -u "$GLOBAL_NEXUX_BASICAUTH" -v

```
