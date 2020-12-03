---
title: Hello World
date: 2020-11-01 15:20:18
tags:
  - 生活
  - 折腾
categories:
  - 环境配置
---
> 满堂花醉三千客，一剑霜寒十四州

## hexo，初次见面，请多多关照

> Test部署更新

> Test自动部署

### 解决tomcat LocalHostlog乱码

##### 解决方法



> 描述：server即服务器日志输出中文正常
>
> localHostlog日志输出异常

> **把tomcat conf下的logging.properites属性里的所有uft-8改为GBK**



> 
>
> 网上帖子说改tomcat conf下的logging.properites
>
> `java.util.logging.ConsoleHandler.encoding = GBK`
>
> 由utf-8修改为gbk，但是本来就是gbk
>
> 按照其他诸如修改idean里tomcat配置加上编码=UTF-8,反而让原来正常的server日志出现乱码~~