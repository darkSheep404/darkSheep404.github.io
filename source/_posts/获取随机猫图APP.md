---
title: 随机获取猫图APP
date: 2020-11-19 15:20:18
tags:
  - kotlin
  - java
  - Android
categories:
  - demo
---

[TOC]



## 安卓实验:随机猫图

>  安卓实验:随机猫图:cat:

> 使用技术：RecycleView，Glide

### 运行效果

![Screenshot_20201124_224008_com.demo.catws](https://sheepnote.oss-cn-shenzhen.aliyuncs.com/20201006/20201124224447.jpg)

### 遇到的待解决的问题

* `Oncreate`里给`RecycleView`传入了八张初始图片，并没有按照瀑布流显示，而是排成了一行
* 运行点击函数后才刷新成设置的两列

> 拟解决办法，`oncreate`里面调用一个刷新函数，函数执行`notifyItemChanged(0)`
>
> > 实际未改变，希望他刷新布局
>
> > 并未解决
>
>
> 

* 

* 点击函数里清空了items数组，使用`myadaper?.notifyItemInserted(data.size-1)`报错越界

  使用`myadaper!!.notifyItemChanged(data.size-1)`反而正常

> 猜想，data数组更新了，但是适配器里的数组还保存着原来的数据，调用notify函数后才更新对应位置数据
>
> 所以主页面里按data空数组下标从0开始插入反而报错
>
> **报错日志**
>
> ```verilog
> D/TAG---->得到第 0 个图片链接：: https://24.media.tumblr.com/tumblr_lwwcqwNiyE1r0mbi6o1_500.jpg
> D/TAG---->得到第 1 个图片链接：: https://25.media.tumblr.com/tumblr_ll6oplOJki1qjahcpo1_500.jpg
> D/TAG---->重置了第1 个图片链接：:             
> D/TAG---->得到第 2 个图片链接：: https://25.media.tumblr.com/tumblr_ln7y9yW8hn1qdth8zo1_500.jpg
> D/TAG---->得到第 3 个图片链接：: https://s3.us-west-2.amazonaws.com/cdn2.thecatapi.com/images/PfA5hN4En.jpg
> D/TAG---->得到第 4 个图片链接：: https://s3.us-west-2.amazonaws.com/cdn2.thecatapi.com/images/1XJ21J4EE.jpg
> D/TAG---->得到第 5 个图片链接：: https://s3.us-west-2.amazonaws.com/cdn2.thecatapi.com/images/2B3n7I5u2.jpg
> D/TAG---->得到第 6 个图片链接：: https://s3.us-west-2.amazonaws.com/cdn2.thecatapi.com/images/0xvhPJ8pd.jpg
> D/TAG---->得到第 7 个图片链接：: https://s3.us-west-2.amazonaws.com/cdn2.thecatapi.com/images/6X-Rt4SAF.jpg
> D/Tag: 执行了onBindViewHolder
> D/AndroidRuntime: Shutting down VM
> E/AndroidRuntime: FATAL EXCEPTION: main
>  Process: com.demo.catws, PID: 23693
>  java.lang.IndexOutOfBoundsException: Inconsistency detected. Invalid item position 7(offset:8).state:8 androidx.recyclerview.widget.RecyclerView{abf2cbd VFED..... ......ID -72,246-1155,2109 #7f070083 app:id/recycle}, 
> ```
>
> 

* 初始化的`recycleview`传入空数组时也不能使用`insert`，但可以使用`changed` 。。。。

* **报错日志**

* ```verilog
  D/TAG---->得到第 0 个图片链接：: https://26.media.tumblr.com/Jjkybd3nSk1fx3xfOQ0aNJawo1_500.jpg
  D/TAG---->得到第 1 个图片链接：: https://24.media.tumblr.com/tumblr_lhc5exQHCr1qgnva2o1_500.jpg
  D/TAG---->得到第 2 个图片链接：: https://24.media.tumblr.com/tumblr_m46gb0Yz7l1qz6rxuo1_500.jpg
  D/TAG---->得到第 3 个图片链接：: https://24.media.tumblr.com/tumblr_m5b8gqPHsN1qenqklo1_r1_500.jpg
  D/TAG---->重置了第3 个图片链接：:             
  D/TAG---->得到第 4 个图片链接：: https://25.media.tumblr.com/tumblr_m6fj9jObek1qzbxjgo1_500.jpg
  D/TAG---->重置了第4 个图片链接：:                 
      https://s3.us-west-2.amazonaws.com/cdn2.thecatapi.com/images/L6X35YZPT.jpg
  D/TAG---->得到第 5 个图片链接：: https://s3.us-west-2.amazonaws.com/cdn2.thecatapi.com/images/L6X35YZPT.jpg
  D/TAG---->得到第 6 个图片链接：: https://s3.us-west-2.amazonaws.com/cdn2.thecatapi.com/images/R1_Oy4xPh.jpg
  D/TAG---->得到第 7 个图片链接：: https://s3.us-west-2.amazonaws.com/cdn2.thecatapi.com/images/HJiePCL8P.jpg
  D/TAG: 进入onCreateViewHolder
  D/Tag: 执行了onBindViewHolder
  //...
  D/Tag: 执行了onBindViewHolder
  I/chatty: uid=10161(com.demo.catws) identical 3 lines
  D/AndroidRuntime: Shutting down VM
  E/AndroidRuntime: FATAL EXCEPTION: main
      Process: com.demo.catws, PID: 24957
      java.lang.IndexOutOfBoundsException: Inconsistency detected. Invalid view holder adapter positionMyViewHolder{a05aa7 position=12 id=-1, oldPos=7, pLpos:7 scrap [attachedScrap] tmpDetached not recyclable(1) no parent} androidx.recyclerview.widget.RecyclerView{3aa67fe VFED..... ......ID -72,246-1155,2109 #7f070083 app:id/recycle}
  ```

  

### 业务代码

```kotlin
//点击函数
fun lookCat(v:View) = Thread(){
        val url="https://api.thecatapi.com/api/images/get?format=xml&size=med&results_per_page=8"
        val xmlString = URL(url).readText()
        val parserFactory: XmlPullParserFactory = XmlPullParserFactory.newInstance()
        val parser: XmlPullParser = parserFactory.newPullParser()
        parser.setFeature(XmlPullParser.FEATURE_PROCESS_NAMESPACES, true)
        parser.setInput(StringReader(xmlString))
        var tag: String?
        var text = ""
        var count=0;
        data.clear()
        var bytes:ByteArray?
        var event = parser.eventType
        while (event != XmlPullParser.END_DOCUMENT) {
            tag = parser.name
            when (event) {
                XmlPullParser.TEXT -> text = parser.text
                XmlPullParser.END_TAG -> when (tag) {
                    "url" -> {
                        //把资源链接压进资源数组
                        //图片链接处理
                        Log.d("TAG---->得到第 ${data.size-1} 个图片链接：", text)
                        data.add(text)
                        runOnUiThread {

                            //myadaper?.notifyItemInserted(data.size-1)
                            //此处使用提醒插入报错，使用提醒改变正常运行
                            myadaper!!.notifyItemChanged(data.size-1)
                            Log.d("TAG---->重置了 ${data.size-1} 个图片链接：", text)

                        }
                        
                      
                    }
                }
            }
            event = parser.next()
        }

    }.start()
```



```kotlin
//图片加载函数
override fun onBindViewHolder(holder: MyViewHolder, position: Int) {
        Log.d("Tag","执行了onBindViewHolder")

        Glide.with(myContext).load(items[position]).into(holder.imgview)

    }
```

## 1.0版本：解析获取xml中的一个图片链接并显示

> 未使用布局及图片加载库

### 效果图

> * 点击按钮向api接口请求xml数据
> * 当从xml中解析出图片url地址后左上角显示：
>   “猫猫图正在向您走来”
> * 从url中读取到图片后，显示图片链接与图片到ui上

![image-20201124200224064](https://sheepnote.oss-cn-shenzhen.aliyuncs.com/20201006/20201124200225.png)

### 业务代码

```kotlin
fun lookCat(v:View) = Thread(){
        val url="https://api.thecatapi.com/api/images/get?format=xml&size=med&results_per_page=1"
        //读取并格式化返回的xml
        val xmlString = URL(url).readText()
        val parserFactory: XmlPullParserFactory = XmlPullParserFactory.newInstance()
        val parser: XmlPullParser = parserFactory.newPullParser()
        parser.setFeature(XmlPullParser.FEATURE_PROCESS_NAMESPACES, true)
        parser.setInput(StringReader(xmlString))
        var tag: String?
        var text = ""
        var bytes:ByteArray?
        var event = parser.eventType
        //开始解析返回的xml，此处因为url设置results_per_page=1，返回的xml只有一个image节点即一张图片链接
        while (event != XmlPullParser.END_DOCUMENT) {
            tag = parser.name
            when (event) {
                XmlPullParser.TEXT -> text = parser.text
                XmlPullParser.END_TAG -> when (tag) {
                    "url" ->{
                        imgurl=text
                        timestart.text="猫猫图正在向您走来！"
                        Log.d("MainActivity","已读取完图片链接 :$imgurl ")
                        bytes = URL(text).readBytes()
                        runOnUiThread{
                            var text=imgurl
                            textView.text=text
                            val bitmap = BitmapFactory.decodeByteArray(bytes, 0, bytes.size)
                            Log.d("MainActivity","已读取完图片字节文件：$imgurl ")
                            imageView.setImageBitmap(bitmap)
                            imageView.isEnabled = true
                        }
                    }
                }
            }
            //当xml存在多个子节点时生效，此时暂不生效
            event = parser.next()
        }

    }.start()

}
```