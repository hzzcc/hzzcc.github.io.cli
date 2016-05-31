---
title: React Native 打包失败
date: 2016-05-28 22:17:32
tags: [React Native]
---

一般我们在打包时，下面命令就可以了
```
react-native bundle --minify --entry-file index.ios.js --bundle-output ios/main.jsbundle --platform ios --dev false
```
当遇到以下错误时
```
FATAL ERROR: CALL_AND_RETRY_LAST Allocation failed - process out of memory
```
需要调整node最大空间，这个时候需要指定最初安装react-native-cli的绝对位置,而我的位置是~/git/nvm/versions/node/v5.6.0/bin/react-native，所以命令如下:
```
node --max-old-space-size=4096 ~/git/nvm/versions/node/v5.6.0/bin/react-native bundle --minify --entry-file index.ios.js --bundle-output ios/main.jsbundle --platform ios --dev false
```

