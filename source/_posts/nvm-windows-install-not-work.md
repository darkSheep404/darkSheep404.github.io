---
title: nvm-windows nvm install not work
date: 2025-01-06 23:20:18
categories:
  - 环境配置
post_meta:
  item_text: false
  created_at: true
  updated_at: true
  categories: true
  tags: true 
---
### nvm-windows nvm install not work
毕业后买的新电脑环境不多,最近折腾博客要升级node,想从node换成nvm,结果一直遇到奇怪问题
nvm-window 安装的nvm install success后 nvm list为空`No installations recognized.`,且nvm use 报错  `nvm use 18.16.0`
`activation error: Version not installed.` 

> 2025年1月6日23:45:09

原因如下(1.22版本前装在非C盘会存在此问题),使用更新版本或者装在C盘可解

`nvm debug`可以查看当前版本

```shell
C:\Users\xxx>nvm debug
xxx is not using admin or elevated rights.

Windows Version:        10.0 (Build 22631)

Windows Developer Mode: UNKNOWN

NVM4W Version:          1.2.0
```

在[nvm list shows node version but nvm use claims its not installed](https://github.com/coreybutler/nvm-windows/issues/979)中发现

https://github.com/coreybutler/nvm-windows/wiki/Common-Issues

原怀疑 nodejs未卸载干净, 但是全局搜nodejs文件夹未发现安装,且node命令未识别,但是发现形似nvm/local/tmp的文件下有下载的`node 18.16.0`文件夹

> NVM for Windows used Go's [os.Rename](https://pkg.go.dev/os#Rename) to move files from temporary directories to the final installation directory. Thanks to explorations with [@thadguidry](https://github.com/thadguidry) in [#1198](https://github.com/coreybutler/nvm-windows/issues/1198) and [#1206](https://github.com/coreybutler/nvm-windows/issues/1206), we realized this function fails silently when attempting to move files across different hard drive volumes (like `C:\` and `E:\`). NVM for Windows uses Go's wonderful support for temporary directories to process installations, which use the primary volume (usually `C:\`). If you are using a different volume for your NVM for Windows installation, it would be subject to this nuance of the `os.Rename` method.
> NVM for Windows使用Go的[os. config](https://pkg.go.dev/os#Rename)将文件从临时目录移动到最终安装目录。感谢[@thadguidry](https://github.com/thadguidry)在[#1198](https://github.com/coreybutler/nvm-windows/issues/1198)和[#1206](https://github.com/coreybutler/nvm-windows/issues/1206)中的探索，我们意识到当尝试在不同的硬盘驱动器卷（如`C:\`和`E:\`）中移动文件时，此功能会默默失败。Windows版NVM使用Go对临时目录的出色支持来处理安装，该安装使用主卷（通常是`C:\`）。如果您正在使用不同的卷进行NVM for Windows安装，则会受到`os.Rename`方法的这种细微差别的影响。
>
> At the time of this writing, a preliminary fix has been created. It is scheduled for release in v1.2.2 after we finalize testing. In the interim, users will need to install NVM for Windows on their C drive.