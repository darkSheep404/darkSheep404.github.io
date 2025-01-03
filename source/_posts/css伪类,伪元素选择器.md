---
date: 2024-05-08 12:20:18
title: css伪类,伪元素选择器
createdTime: 22-11-16 (星期三) 23:42
updateTime: 23-03-10 (星期五) 16:54
---
#css 
`::`即为伪类选择器

## 伪元素选择器

> ## 7. 伪元素选择器
>
> `伪元素，表示页面中一些特殊的并不真实的存在的元素（特殊的位置）`
>
> 伪元素使用`::`开头
>
> * `::first-letter` 表示第一个字母
>
> * `::first-line` 表示第一行
>
> * `::selection` 表示选中的内容
>
> * `::before` 元素的开始
>
> * `::after` 元素的最后
>
> * `::before`和`::after` 必须结合`content`属性来使用

### 代码片段

```css
/* 段落首字母设置大小为30px */
p::first-letter{
    font-size: 30px;
}

/* 段落第一行设置为黄色背景 */
p::first-line{
    background-color: yellow;
}

/* 段落选中的部分变绿色 */
p::selection{
    background-color: green；
}

/* div前加上内容 */
div::before{
    content: 'BEFORE';
    color: red;
}

/* div后加上内容 */
div::after{
    content: 'AFTER';
    color: blue;
}
```



## 伪类:元素特殊状态一般`:` 开头

>
>
>伪类一般情况下都是使用`:`开头
>
>
>
>* `:first-child` 第一个子元素
>
>* `:last-child` 最后一个子元素
>
>* `:nth-child()` 选中第n个子元素 
>
>* * n：第n个，n的范围0到正无穷
>
>* * 2n或even：选中偶数位的元素
>
>* * 2n+1或odd：选中奇数位的元素
>
>以上这些伪类都是根据所有的子元素进行排序的
>
>* `:first-of-type` 同类型中的第一个子元素
>
>* `:last-of-type` 同类型中的最后一个子元素
>
>* `:nth-of-type()` 选中同类型中的第n个子元素
>
>这几个伪类的功能和上述的类似，不同点是他们是在同类型元素中进行排序的
>
>* `:not()`否定伪类，将符合条件的元素从选择器中去除
>* `:link` 未访问的链接
>
>* `:visited` 已访问的链接 
>
>* * 由于隐私的原因，所以`visited`这个伪类只能修改链接的颜色
>
>* `:hover` 鼠标悬停的链接
>
>* `:active` 鼠标点击的链接

### 代码片段

```css
/* ul下所有li，黑色 */
ul>li {
    color: black;
}

/* ul下第偶数个li，黄色 */
ul>li:nth-child(2n) {
    color: yellow;
}

/* ul下第奇数个li，绿色 */
ul>li:nth-child(odd) {
    color: green;
}

/* ul下第一个li，红色 */
ul>li:first-child {
    color: red;
}

/* ul下最后一个li，黄色 */
ul>li:last-child {
    color: orange;
}
```



```css
/* unvisited link */
a:link {
  color: red;
}

/* visited link */
a:visited {
  color: yellow;
}

/* mouse over link */
a:hover {
  color: green;
}

/* selected link */
a:active {
  color: blue;
}
```

