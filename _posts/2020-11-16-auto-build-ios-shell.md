---
layout: blog_default
title:  "自动构建ios APP脚本"
---

# 自动构建ios APP脚本

``` shell
    #! /bin/bash -ilex

    set -e

    buildType=$1
    basePath=$WORKSPACE/mobile
    configPath=$WORKSPACE/build
    IpaName='.*****.'
    IpaOutputPath='platforms/ios/build/device'
    readonly GLOBAL_NEXUX='https://nexus.*****.net'
    readonly GLOBAL_NEXUX_BASICAUTH='.*****.'
    echo $IpaName
    echo 'basePath===>'$basePath
    echo 'configPath===>'$configPath
    echo 'WORKSPACE===>'$WORKSPACE
    echo 'IpaOutputPath===>'$IpaOutputPath

    publishDir=./${npmBuildDist:-www}
    echo 'publishDir===>'$publishDir

    getVersion() {
        cd $basePath
        packageVer=$(cat package.json | sed 's/,/\n/g' | grep '"version' | cut -b 15-19)
        echo "${npmPackageJson:-package.json} ------version: $packageVer"
        if [[ -n $packageVer ]]; then
            return
        fi
        packageVer=1.0.0
    }

    iosPublish() {
        echo "ios ipa building start"
        cd $basePath
        # if [[ ! -d platforms/ios ]]; then
        yes | cordova9 platform add ios
        # fi
        # rm -rf platforms/ios/$IpaName.xcodeproj
        # cp -R $configPath/$IpaName.xcodeproj platforms/ios

        ipafile=./$IpaOutputPath/$IpaName.ipa
        ipaname=$IpaName-$buildType-$packageVer.ipa
        echo $ipaname
        cp $configPath/ios/ExportOptions.plist $basePath/platforms/ios/ExportOptions.plist
        if [ $? != 0 ]; then
            echo "could not find plist file, build faild..."
            exit 1
        fi
        security set-key-partition-list -S apple-tool:,apple: -s -k "123456" ~/Library/Keychains/login.keychain-db
        if [ $? != 0 ]; then
            echo "unlock keychain faild, build faild..."
            exit 1
        fi
        chmod 777 $basePath/hooks/before_build.sh
        if [ "${buildType}" == 'prod' ]; then
            echo "building prod ipa"
            artifactDir='fcloudapp-release/ios'
            cordova9 build ios --device --release --buildConfig=$configPath/ios/build.json --verbose
        else
            echo "building dev ipa"
            artifactDir='fcloudapp-dev/ios'
            cordova9 build ios --device --debug --buildConfig=$configPath/ios/build.json --verbose
        fi
        echo "ios ipa building end"

        cp $ipafile $publishDir/$ipaname
        cd ./$publishDir
        echo $artifactDir
        curl -X POST "$GLOBAL_NEXUX/service/rest/v1/components?repository=artifact" -H "accept: application/json" -H "Content-Type: multipart/form-data" -F "raw.directory=fcloud-artifact/$artifactDir" -F "raw.asset1=@$ipaname" -F "raw.asset1.filename=$ipaname" -u "$GLOBAL_NEXUX_BASICAUTH" -v
        echo "ios ipa upload ok"
    }

    getVersion
    iosPublish

```
