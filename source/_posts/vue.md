---
title: vue代码逻辑
date: 2019-1-23  
---

###  vue代码基本逻辑结构
 ![image](https://pic4.zhimg.com/v2-fab4b9d1bb38ec6d1d515a9d260f4737_b.png)
**index.html**

```
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <title>vue-demo</title>
    <style>
      body {
        margin: 0;
        font-family: Microsoft Yahei;
        background: #f5f8fb;
      }
    </style>
  </head>
  <body>
    <div id="app"></div>
    <script src="/dist/build.js"></script>
  </body>
</html>
```

index.html是整个项目的入口文件，它只加载build.js文件。build.js是除了index.html外所有文件经过webpack打包生成的。所以页面加载时浏览器只加载index.html和build.js两个文件。

**main.js**


```
import Vue from 'vue' /*引入Vue.js核心库*/
import App from './App.vue' /*引入App.vue父组件*/

/*创建一个Vue的根实例*/
new Vue({
  el: '#app',
  render: h => h(App)
})
```
main.js是webpack的入口文件，webpack会从它入手打包成最后的build.js。在main.js里我们只需要创建一个Vue的根实例并把App.vue渲染到页面上。

**App.vue**


```
<template>
  <div id="app"><!-- 模板内只能有一个根节点 -->
    <!-- 监听子组件的change事件 -->
    <my-navbar class="my-navbar" v-on:change="onChangeData"></my-navbar>
    <!-- 使用prop向子组件传递数据 -->
    <my-table class="my-table" v-bind:datas="lists"></my-table>
  </div>
</template>

<script>
/*引入子组件*/
import Navbar from './component/navbar.vue';
import Table from './component/table.vue';
import List from './model/data.vue';
export default {
  /*定义该组件模板中需要用的变量*/
  data () {
    return {
      lists: List.data()["2016Q1"]
    }
  },
  /*子组件注册*/
  components: {
    'my-navbar': Navbar,
    'my-table': Table
  },
  /*定义该组件模板中需要触发的方法*/
  methods: {
    onChangeData: function(id) {
      /* 直接修改数组，Vue.js不能检测到数据的变化，因此下面这行代码无效 */
      // this.lists = List.data()[id];
      /* 通过$set方法修改数组 */
      for (var i = 0; i < this.lists.length; i++) {
        this.$set(this.lists, i, List.data()[id][i]);
      };
    }
  }
}
</script>

<style lang="scss" scoped>
.my-table {
  margin: 90px 7px 0 7px;
}
</style>
```
父组件App.vue统筹管理navbar.vue（导航栏）、table.vue（表格）、data.vue（数据）三个子组件。父组件需要做三件事情：一、从navbar.vue监听获取日期数据；二、根据日期数据从data.vue获取相应的表格数据；三、把表格数据传递给表格。

**navbar.vue**
```<template>
  <div id="navbar">
    <span>XX报表</span>
    <img v-bind:src="img" v-on:click="toggle" />
    <div class="list" v-show="listShow">
      <div class="item" v-on:click="changedata('2016Q1')">2016Q1</div>
      <div class="item" v-on:click="changedata('2016Q2')">2016Q2</div>
    </div>
  </div>
</template>

<script>
import funnel from "../img/funnel.png";
export default {
  data () {
    return {
      img: funnel,
      listShow: false
    }
  },
  methods: {
    /* 控制菜单的开合 */
    toggle: function() {
      this.listShow = !this.listShow;
    },
    /* 向父组件发送事件change，并捎带参数id，也就是选择的时间 */
    changedata: function(id) {
      this.$emit('change', id);
      this.listShow = false;
    }
  }
}
</script>

<style lang="sass" scoped>
#navbar {
  position: relative;
  width: 100%;
  height: 40px;
  line-height: 40px;
  text-align: center;
  background: #37363b;
  color: #fff;
  img {
    position: absolute;
    width: 20px;
    height: 20px;
    top: 10px;
    right: 10px;
  }
  .list {
    position: absolute;
    right: 5px;
    top: 40px;
    width: 70px;
    height: 80px;
    background: #37363b;
  }
}
</style>
```
navbar.vue的职责是当点击下拉框向App.vue传递日期数据。这其实是一个由子组件向父组件传递数据的过程。在Vue.js里，父组件给子组件传递数据可以简单地用prop传递，但prop的传递是单向的，只能由父组件向子组件传递。子组件向父组件的数据传递要由子组件$emit一个事件，然后父组件去监听这个事件再进行一系列的操作。

**table.vue**
```
<template>
  <div id="table">
    <table>
      <thead>
      	<tr>
      	  <td>频道</td>
      	  <td>当日vv量(千)</td>
      	  <td>日环比</td>
      	  <td>周同比</td>
      	</tr>
      </thead>
      <tbody>
        <tr v-for="item in items">
          <td>{{ item.channel }}</td>
          <td>{{ item.vv }}</td>
          <td>{{ item.dayrate }}</td>
          <td>{{ item.weekrate }}</td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
export default {
  /* 父组件传来的数据 */
  props: ['datas'],
  data () {
    return {
      items: this.datas,
    }
  }
}
</script>

<style lang="sass" scoped>
table {
  width: 400px;
  border-spacing: 0;
  border-collapse: collapse;
}
td {
  text-align: center;
  border: 1px solid #ccc;
}
</style>
```
table.vue通过prop获取父组件传递过来的数据。v-for，双大括号等一些语法跟angularjs很像。

**data.vue**
```
<script>
export default {
  data () {
    return {"2016Q1": [
      {channel: '娱乐', vv: '17,980', dayrate: '+292%', weekrate: '+5%'},
      {channel: '游戏', vv: '48,133', dayrate: '+48%', weekrate: '+21%'},
      {channel: '动漫', vv: '189,342', dayrate: '+39%', weekrate: '+0%'},
      {channel: '综艺', vv: '106,430', dayrate: '+35%', weekrate: '+12%'}
    ],
    "2016Q2": [
      {channel: '娱乐', vv: '43,910', dayrate: '+414%', weekrate: '+77%'},
      {channel: '游戏', vv: '34,173', dayrate: '+28%', weekrate: '+11%'},
      {channel: '动漫', vv: '89,562', dayrate: '+59%', weekrate: '+29%'},
      {channel: '综艺', vv: '96,189', dayrate: '+37%', weekrate: '+0%'}
    ]}
  }
}
</script>
```

## 结尾
这个demo的model里写的都是假数据，实际项目中这里可以写调取后台接口的方法。

通过这个demo让大家先对Vue.js的使用有一个基本的印象，后续大家可以自己用Vue.js做一些更复杂的项目来对它有一个更深层次的理解。
