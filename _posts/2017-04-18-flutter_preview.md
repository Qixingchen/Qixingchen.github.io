---
layout: post
title: Flutter 预览介绍
---
# Flutter

## Flutter 介绍

Flutter 是一个使用同一代码构建高性能,高保真的 iOS 与 Android 应用的 SDK.
目前由 Google 主持开发.

### Flutter 设备要求

#### 开发设备
- 支持 win Mac Linux 三端.集成 iOS 应用要求 Xcode.

#### 终端设备
- iOS 端要求 64 位设备.iOS 8 及以上
- Android 端要求 Android Jelly Bean, v16, 4.1.x 及以上(目前 Android 端开发时最低支持 Kitkat, v19, 4.4.x )
- 不推荐在平板上使用,目前不支持 wear
- 目前不支持 web 与 桌面端

### Flutter 优势

1. 拥抱差异.包括 iOS Android 两端在滚动行为,图标与排版上的差异都可以使用同一代码来实现.
1. 高性能.基于底层代码(Android 上为 C++ with NDK,iOS 上为 C++ with LLVM),不使用 WebView 渲染,不涉及翻译.能够保证 60 fps 的绘制祯率.
1. 可以与原生代码互调.
1. 提供了大部分 Material Design 控件的实现(甚至比 Android Design Support 实现的更多).
1. 足够小. SDK 大小在两端都在 8MB 左右,可以接受这样的体积增加.


### Flutter 劣势

1. 仍在 Preview 阶段,还未释出 1.0
1. 不支持 3D 应用(不过我们也不做)


