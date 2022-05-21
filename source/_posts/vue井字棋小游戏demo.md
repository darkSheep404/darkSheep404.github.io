---
title: vue井字棋小游戏
date: 2020-11-26 15:33:18
tags:
  - demo
  - vue
categories:
  - 前端
post_meta:
  item_text: false
  created_at: true
  updated_at: true
  categories: true
  tags: true  
---

### 井字棋

`使用vue`

#### [本站预览链接](https://darksheep.xyz/game01/)
#### GitHub单网页预览链接

> 待补充

**tips**

> 网页制作之初是单独作为一个网页设计，现在挂靠在博客的右边栏
>
> 胜利后的布局出现稍微错乱

### 一个旋转的太极

#### [本站预览链接](https://darksheep.xyz/taiji/)

`使用html css`

> 利用html和css制作的一个简单的太极单页面
>
> * 不清楚是否也会同本页面发生冲突

### vue打包发布

 > vue打包成html
  >
  > `yarn build` or `npm run build`
  >
  > vue打包dist目录下index.html打开空白
  >
  > 根目录下新建`vue.config.js`，填入以下代码
  >
  > > vue cli3.0后此文件隐藏，不在根目录下，可以自己手动新建文件覆盖相应配置
  >
  > ```js
  > module.exports = {
  >     lintOnSave: true,
  > 
  >     publicPath: './',
  >     outputDir: 'dist',
  >     assetsDir: 'static'
  > }
  > 
  > ```
  >
  > 



### 井字棋demo源码

#### app.vue

```vue
<template>
  <div id="app">
    <div id="res" v-if="win">游戏结束，获胜者是：{{ winner }}</div>
    <div class="row">
      <cell @click="onclickCell(0, $event)" :n="n" />
      <cell @click="onclickCell(1, $event)" :n="n" />
      <cell @click="onclickCell(2, $event)" :n="n" />
    </div>
    <div class="row">
      <cell @click="onclickCell(3, $event)" :n="n" />
      <cell @click="onclickCell(4, $event)" :n="n" />
      <cell @click="onclickCell(5, $event)" :n="n" />
    </div>
    <div class="row">
      <cell @click="onclickCell(6, $event)" :n="n" />
      <cell @click="onclickCell(7, $event)" :n="n" />
      <cell @click="onclickCell(8, $event)" :n="n" />
    </div>
  </div>
</template>

<script>
import cell from "./cell";
export default {
  data() {
    return {
      cells: [
        [1, 1, 1],
        [1, 1, 1],
        [1, 1, 1],
      ],
      /*  
      写成如下没带逗号出现bug，影响到n的赋值，但是如下代码null批量替换成1后代码也可以运行
      cells: [[null, null, null] 
      [(null, null, null)]

      [(null, null, null)]], */
      n: 0,
      win: false,
      winner: null,
    };
  },
  components: { cell },
  methods: {
    onclickCell(i, text) {
      console.log(`${i}号被点击，内容是${text}`); //插入变量是用反单引号括起来
      this.cells[Math.floor(i / 3)][i % 3] = text;
      this.n = this.n + 1;
      this.tell();
      if (this.win === true) {
        //此处误写掉括号，一直判定成true
        this.winner = text;
        console.log(`获胜者是${text}`);
      } else {
        console.log("胜负未定");
      }
    },
    tell() {
      const cells = this.cells;
      for (let i = 0; i < 2; i++) {
        if (
          cells[i][0] === cells[i][1] &&
          cells[i][0] === cells[i][2] &&
          cells[i][0] !== 1
        ) {
          this.win = true;
        }
      }
      for (let j = 0; j < 2; j++) {
        if (
          cells[0][j] === cells[1][j] &&
          cells[0][j] === cells[2][j] &&
          cells[0][j] !== 1
        ) {
          this.win = true;
        }
      }
      if (
        cells[0][0] !== 1 &&
        cells[2][2] == cells[0][0] &&
        cells[1][1] === cells[2][2]
      ) {
        this.win = true;
      }
      if (
        cells[0][2] !== 1 &&
        cells[1][1] == cells[2][0] &&
        cells[0][2] === cells[1][1]
      ) {
        this.win = true;
      }
    },
  },
};
</script>

<style>
.row {
  display: flex;
  justify-content: center;
  align-items: center;
}
#res {
  border: 1px solid red;
  background: lightblue;
  position: absolute;
  height: 374px;
  width: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
}
</style>

```

#### cell.vue

```vue
<template>
  <div id="cell" v-on:click="onclickself">
    <template v-if="a">{{ text }}</template>
    <template v-else></template>
  </div>
</template>

<script>
//node中向外暴露成员：moudule.exports={}和exports，导入模块：var 名称=require('标识符')
//ES6中导入：import xxx form xx，导出：export default和export 向外暴露
//两种不能混用
//export导出，导入时要加{}:按需导出，export 可以向外导出多个成员，import时如果不需要，可以不在{}中定义
//export导出的成员，导入时必须严格按照名称{}按需接受 想换名字可以使用as 起别名
// export default 导出的成员，可以使用任意变量接收，导入时不需要加{} 一个文件中，export可以有多个，export default只可以有一个
export default {
  //父组件使用v-bind或简化指令传递数据给子组件
  //子组件通过props接收，只能读不能更改
  props: ["n"],
  data() {
    //:() => {
    //function(){}
    //return (x = 1);
    return { a: false, text: "o" };
  },
  methods: {
    onclickself() {
      if (this.a === true) {
        return;
      } else {
        this.a = true;
        this.text = this.n % 2 === 0 ? "x" : "o";
        //父组件通过v-on把其方法引用传递给子组件，子组件通过$emit,调用父组件方法，并把数据传给父组件使用
        this.$emit("click", this.text);
      }
    },
  },
};
</script>

<style>
#cell {
  border: 1px solid black;
  width: 100px;
  height: 100px;
  display: flex;
  justify-content: center;
  align-items: center;
  font-size: 80px;
}
</style>

```

#### main.js

```js
import Vue from 'vue'
import App from './App.vue'

Vue.config.productionTip = false

new Vue({
  render: h => h(App),
}).$mount('#app')

```





```html
<!DOCTYPE html>
<html lang="zh">

<head>
  <meta charset="UTF-8">
  <title>doucument</title>
  <style>
    @keyframes x {
      from {
        transform: rotate(0deg);
      }

      to {
        transform: rotate(360deg);
      }

    }

    * {
      /*background: #eee;*/
      box-sizing: border-box;

      padding: 0;
      margin: 0;
    }

    body {
      background: #eee;
    }

    #八卦 {
      /*border:1px solid red;*/
      width: 400px;
      height: 400px;
      border-radius: 200px;
      position: relative;
      overflow: hidden;
      animation: x 1s infinite linear;
      box-shadow: 0px 0px 3px 0px rgba(0, 0, 0, 0.75);
    }

    #八卦>div:nth-child(2) {
      /*border:10px solid green;*/
      width: 50%;
      height: 100%;
      position: absolute;
      right: 0;
      background: white;

    }

    #八卦>div:first-child {
      /*border:10px solid yellow;*/
      width: 50%;
      position: absolute;
      height: 100%;
      left: 0;
      background: black;
    }

    #八卦>div:nth-child(3) {
      /*border:10px solid green;*/
      width: 50%;
      height: 50%;
      border-radius: 200px;
      position: absolute;
      /*right: 0;此处多写了一个右边，但是好像没影响*/
      left: 50%;
      background: black;
      margin-left: -100px;

    }

    #八卦>div:nth-child(4) {
      /*border:10px solid green;*/
      width: 50%;
      height: 50%;
      border-radius: 200px;
      position: absolute;
      right: 50%;
      background: white;
      margin-right: -100px;
      bottom: 0;

    }

    #八卦>div:nth-child(5) {
      /*上小白圆*/
      /*border:10px solid green;*/
      width: 12.5%;
      height: 12.5%;
      border-radius: 200px;
      position: absolute;
      left: 50%;
      background: white;
      margin-left: -6.25%;
      margin-top: 6.25%;
      top: 12.5%;

    }

    #八卦>div:nth-child(6) {
      /*下小白圆*/
      /*border:10px solid green;*/
      width: 12.5%;
      height: 12.5%;
      border-radius: 200px;
      position: absolute;
      left: 50%;
      background: black;
      margin-left: -6.25%;
      margin-bottom: 6.25%;
      bottom: 12.5%;
    }

    #八卦-wrapper {
      border: solid 1px red;
      display: flex;
      justify-content: center;
      align-items: center;
      flex-direction: column;
      height: 100vh;
    }

    #txt {
      margin-top: 1em;
    }
  </style>
</head>

<body>
  <div id="八卦-wrapper">
    <div id="八卦">
      <div>1</div>
      <div>2</div>
      <div>3</div>
      <div>4</div>
      <div></div>
      <div></div>

    </div>
    <div id="txt">道可道，非常道</div>
  </div>
</body>

</html>
```

