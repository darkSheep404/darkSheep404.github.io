---
date: 2024-05-02 12:20:18
title: 结合el-dialog使用.sync和vuex
createdTime: 22-12-09 (星期五) 16:50
updateTime: 23-03-10 (星期五) 16:49
---
### 背景
> 需要每次刷新页面或者切换页面时 
> 调用api 查询是否需要弹窗
> 并弹出弹窗

#### 监听 vue刷新或切换页面
```js
VueRouter.afterEach(() => {  
  if (VueRouter.app.$store.getters.token) {  
    console.log('%c用户刷新', 'color:white;background:blue;font-weight: bold;')
    //check中 调用set 将返回值设置进vuex中  
    checkNeedDialog()  
  }  
})
```
### dialog .sync 与vuex
>当需要在子组件中更新父组件传入的值时

子组件应通过$emit提交事件,传入要更新的属性和新值
`this.$emit('update:title', newTitle)`
父组件监听事件并更新值
`v-on:update:title="doc.title = $event`
父组件中简写
`v-bind:title.sync` 即`:title.sync`
### 实际实现

#### 弹窗组件中
> 控制弹窗是否可见的变量
> 由父组件传入
> 子组件通过计算属性 获取值

```vue
<el-dialog  
  :visible="dialogVisible"  >
<el-button @click="dialogVisible = false">{{ $t('gotIt') }}</el-button>
</el-dialog>
```

```js
computed: {  
  dialogVisible: {  
    get() {  
      return this.visible  
    },  
    set(val) {  
      this.$emit('update:visible', val)  
    }  
  }  
}
```

#### 父组件使用时

> 组件上设置:visible.sync
```vue
<share-notify :visible.sync = "isShowShareDialog"/>

```
> 绑定的变量为计算属性
> 以上两处的计算属性因为除了get以外还需要set
> 所以要写成`xxx:{}`的格式
> 只有get的计算属性 则可以写成 `xxx(){return xxx}`的简写


```js
isShowShareDialog: {  
  get() {  
    return this.$store.state.user.isShowEquoteShareDialog  
  },  
  set() {  
    this.$store.dispatch('confirmEquoteShareDialog')  
  }  
}
```
#### 改进
> 不过此例中,结合上述代码可以发现
> 不需要由父组件根据vuex中值 更新子组件的可见状态
> 直接子组件的`visible` 与vuex中数据绑定即可
### 参考链接

[:visible.sync 的作用_卡卡西Sensei的博客-CSDN博客_:visible.sync](https://blog.csdn.net/zjpjay/article/details/113992083)
[.sync修饰符 — Vue.js](https://v2.cn.vuejs.org/v2/guide/components-custom-events.html#sync-%E4%BF%AE%E9%A5%B0%E7%AC%A6)
