# Ionic 下开发跨平台应用


___

本项目将涵盖两个方面的内容：Android APP与桌面应用的开发，采用HTML5方式。

你需要掌握HTML/JAVASRIPT/CSS基础、TypeScript语言、Angular2框架等方面的知识。然后在Ionic/Cordova/Electron的支持下，就可以进行跨平台应用的开发。


___

在这里阅读本书： [hrybrid-dev.xfoss.com](https://hybrid-dev.xfoss.com)。

在这里Fork 本项目：[gnu4cn/html5AndroidAppHowtos](https://github.com/gnu4cn/html5AndroidAppHowtos)

> 如你觉得本项目对你有所帮助，请考虑 [捐赠译者](https://github.com/gnu4cn/buy-me-a-coffee)。


## Android 虚拟机中调试消息获取

使用 Logcat 功能，可以获取到物理机或虚拟机中的 `console.log` 输出。

```console
$ adb devices
List of devices attached
emulator-5554	device
$ adb -s emulator-5554 logcat | grep chromium
```

## 目录

[开发环境搭建(Android APP)](00_dev_environment.md)
