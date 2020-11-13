---
layout: blog_default
title:  "基于cordova的安卓APP基础镜像"
tag: [docker, android]
---

# 基于cordova的安卓APP基础镜像

**镜像分为三层，分别是**

* java环境基础镜像
* android环境基础镜像
* cordova环境基础镜像

### java环境基础镜像(底层是xenial的ubuntu)

**jdk版本： 1.8**  
**镜像名称： javabase**

``` sh
FROM ubuntu:xenial

MAINTAINER LINMAN 

ENV DEBIAN_FRONTEND=noninteractive \
    TERM=xterm

LABEL maintainer="LINMAN" \
  org.label-schema.schema-version="1.0.0-rc1" \
  org.label-schema.name="base/java" \
  org.label-schema.description="Simple Java Docker image (used as base image)" 

# required to use add-apt-repository
RUN buildDeps='software-properties-common'; \
  set -x && \
  apt-get update && apt-get install -y $buildDeps --no-install-recommends && \
  add-apt-repository ppa:openjdk-r/ppa -y && \
  apt-get update -y && \
  apt-get install -y openjdk-8-jdk && \
  java -version && \
  rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* && \
  apt-get purge -y --auto-remove $buildDeps && \
  apt-get autoremove -y && apt-get clean

ENV JAVA_HOME /usr/lib/jvm/java-8-openjdk-amd64

```

### android环境基础镜像

**android sdk： 28**  
**镜像名称： androidbase**

|序号|名称|默认值|
|:--------|:-------:|--------:|
|1|GRADLE_VERSION|3.3|  

``` sh
FROM 私有仓/javabase:1.0

MAINTAINER LINMAN

RUN apt-get -qq update \
    && apt-get -qq install -y wget curl zip
	
ARG GRADLE_VERSION=3.3

WORKDIR /usr/share

RUN wget -q https://services.gradle.org/distributions/gradle-${GRADLE_VERSION}-bin.zip \
    && unzip -q gradle-${GRADLE_VERSION}-bin.zip \
	&& mv gradle-${GRADLE_VERSION} gradle \
    && rm gradle-${GRADLE_VERSION}-bin.zip
ENV GRADLE_HOME="/usr/share/gradle"

ENV ANDROID_SDK_URL="https://dl.google.com/android/repository/sdk-tools-linux-4333796.zip" \
    ANDROID_BUILD_TOOLS_VERSION=28.0.3 \
    ANDROID_APIS="android-28" \
    ANDROID_HOME="/opt/android" \
	ANDROID_SDK_ROOT="/opt/android/platforms/${ANDROID_APIS}"
ENV PATH $PATH:${ANDROID_HOME}/tools:${ANDROID_HOME}/platform-tools:${ANDROID_HOME}/build-tools/${ANDROID_BUILD_TOOLS_VERSION}:${GRADLE_HOME}/bin

WORKDIR /opt

# Installs Android SDK
RUN mkdir android && cd android \
    && wget -O tools.zip ${ANDROID_SDK_URL} \
    && unzip tools.zip && rm tools.zip

RUN mkdir /root/.android && touch /root/.android/repositories.cfg \ 
    && yes | android update sdk -a -u -t platform-tools,${ANDROID_APIS},build-tools-${ANDROID_BUILD_TOOLS_VERSION} \
    && chmod a+x -R ${ANDROID_HOME} \
    && chown -R root:root ${ANDROID_HOME} \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* \
    && apt-get autoremove -y \
    && apt-get clean

```

### cordova环境基础镜像

**cordova： 9.0**  
**镜像名称： cordovabase**

|序号|名称|默认值|
|:-:|:-:|:-:|
|1|BASE_IMAGE_VERSION|28|
|2|BASE_IMAGE_TAG|1.0|

``` sh
ARG BASE_IMAGE_VERSION=28
ARG BASE_IMAGE_TAG=1.0

FROM docker.flexem.com/flexem-fcloud/androidbase${BASE_IMAGE_VERSION}:${BASE_IMAGE_TAG}

MAINTAINER LINMAN

ENV NODEJS_VERSION=12.10.0 \
    PATH=$PATH:/opt/node/bin

WORKDIR "/opt/node"

RUN apt-get update && apt-get install -y curl git ca-certificates --no-install-recommends && \
    curl -sL https://nodejs.org/dist/v${NODEJS_VERSION}/node-v${NODEJS_VERSION}-linux-x64.tar.gz | tar xz --strip-components=1 && \
    rm -rf /var/lib/apt/lists/* && \
    apt-get clean

RUN npm install -g yarn

ENV CORDOVA_VERSION 9.0.0

WORKDIR "/tmp"

RUN npm i -g --unsafe-perm cordova@${CORDOVA_VERSION}

```
