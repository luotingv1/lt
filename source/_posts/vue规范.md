---
title: vue规范  
date: 2019-1-30  
---
# vue 开发规范

## 组件数据必要
组件的 data 必须是一个函数。
当在组件中使用 data 属性的时候 (除了 new Vue外的任何地方)，它的值必须是返回一个对象的函数。
- 详解
当 data 的值是一个对象时，它会在这个组件的所有实例之间共享。想象一下，假如一个 TodoList 组件的数据是这样的：
```
data: {
  listTitle: '',
  todos: []
}
```
我们可能希望重用这个组件，允许用户维护多个列表(比如分为购物、心愿单、日常事务等)。这时就会产生问题。因为每个组件的实例都引用了相同的数据对象，更改其中一个列表的标题就会改变其它每一个列表的标题。增删改一个待办事项的时候也是如此。

取而代之的是，我们希望每个组件实例都管理其自己的数据。为了做到这一点，每个实例必须生成一个独立的数据对象。在 JavaScript 中，在一个函数中返回这个对象就可以了：
```
data: function () {
  return {
    listTitle: '',
    todos: []
  }
}
```
反例
```
Vue.component('some-comp', {
  data: {
    foo: 'bar'
  }
})
export default {
  data: {
    foo: 'bar'
  }
}
```
好例子
```
Vue.component('some-comp', {
  data: function () {
    return {
      foo: 'bar'
    }
  }
})
// In a .vue file
export default {
  data () {
    return {
      foo: 'bar'
    }
  }
}
// 在一个 Vue 的根实例上直接使用对象是可以的，
// 因为只存在一个这样的实例。
new Vue({
  data: {
    foo: 'bar'
  }
})
```
## Prop 定义应该尽量详细。

在你提交的代码中，prop 的定义应该尽量详细，至少需要指定其类型。
- 详解
细致的 prop 定义有两个好处：
1. 它们写明了组件的 API，所以很容易看懂组件的用法；
2. 在开发环境下，如果向一个组件提供格式不正确的 prop，Vue 将会告警，以帮助你捕获潜在的错误来源。
反例
// 这样做只有开发原型系统时可以接受
```
props: ['status']
```
好例子
```
props: {
  status: String
}
// 更好的做法！
props:{
    status:Array,
    required:true,
    default:()=>{
        return []
    }
}
props: {
  status: {
    type: String,
    required: true,
    validator: function (value) {
      return [
        'syncing',
        'synced',
        'version-conflict',
        'error'
      ].indexOf(value) !== -1
    }
  }
}
```
## 总是用 key 配合 v-for。
在组件上总是必须用 key 配合v-for，以便维护内部组件及其子树的状态。甚至在元素上维护可预测的行为，比如动画中的对象固化 (object constancy)，也是一种好的做法。
 - 详解
假设你有一个待办事项列表：
```
data: function () {
  return {
    todos: [
      {
        id: 1,
        text: '学习使用 v-for'
      },
      {
        id: 2,
        text: '学习使用 key'
      }
    ]
  }
}
```
然后你把它们按照字母顺序排序。在更新 DOM 的时候，Vue 将会优化渲染把可能的 DOM 变动降到最低。即可能删掉第一个待办事项元素，然后把它重新加回到列表的最末尾。
这里的问题在于，不要删除仍然会留在 DOM 中的元素。比如你想使用 <transition-group> 给列表加过渡动画，或想在被渲染元素是 input时保持聚焦。在这些情况下，为每一个项目添加一个唯一的键值 (比如 :key="todo.id") 将会让 Vue 知道如何使行为更容易预测。
根据我们的经验，最好始终添加一个唯一的键值，以便你和你的团队永远不必担心这些极端情况。也在少数对性能有严格要求的情况下，为了避免对象固化，你可以刻意做一些非常规的处理。

反例
```
<ul>
  <li v-for="todo in todos">
    {{ todo.text }}
  </li>
</ul>
```
好例子
```
<ul>
  <li
    v-for="todo in todos"
    :key="todo.id"
  >
    {{ todo.text }}
  </li>
</ul>
```
## 避免 v-if 和 v-for 用在一起 必要 
永远不要把 v-if 和 v-for 同时用在同一个元素上。
不要在 v-for 的元素上使用 item 来进行一些操作
一般我们在两种常见的情况下会倾向于这样做：
1. 为了过滤一个列表中的项目 (比如 v-for="user in users" v-if="user.isActive")。在这种情形下，请将 users 替换为一个计算属性 (比如 activeUsers)，让其返回过滤后的列表。
2. 为了避免渲染本应该被隐藏的列表 (比如 v-for="user in users" v-if="shouldShowUsers")。这种情形下，请将 v-if 移动至容器元素上 (比如 ul, ol)。
- 详解
当 Vue 处理指令时，v-for 比 v-if 具有更高的优先级，所以这个模板：
```
<ul>
  <li
    v-for="user in users"
    v-if="user.isActive"
    :key="user.id"
  >
    {{ user.name }}
  </li>
</ul>
```
将会经过如下运算：
```
this.users.map(function (user) {
  if (user.isActive) {
    return user.name
  }
})
```
因此哪怕我们只渲染出一小部分用户的元素，也得在每次重渲染的时候遍历整个列表，不论活跃用户是否发生了变化。
通过将其更换为在如下的一个计算属性上遍历：
```
computed: {
  activeUsers: function () {
    return this.users.filter(function (user) {
      return user.isActive
    })
  }
}
<ul>
  <li
    v-for="user in activeUsers"
    :key="user.id"
  >
    {{ user.name }}
  </li>
</ul>
```
我们将会获得如下好处：
过滤后的列表只会在 users 数组发生相关变化时才被重新运算，过滤更高效。
使用 v-for="user in activeUsers" 之后，我们在渲染的时候只遍历活跃用户，渲染更高效。
解藕渲染层的逻辑，可维护性 (对逻辑的更改和扩展) 更强。
为了获得同样的好处，我们也可以把：
```
<ul>
  <li
    v-for="user in users"
    v-if="shouldShowUsers"
    :key="user.id"
  >
    {{ user.name }}
  </li>
</ul>
```
更新为：
```
<ul v-if="shouldShowUsers">
  <li
    v-for="user in users"
    :key="user.id"
  >
    {{ user.name }}
  </li>
</ul>
```
通过将 v-if 移动到容器元素，我们不会再对列表中的每个用户检查 shouldShowUsers。取而代之的是，我们只检查它一次，且不会在 shouldShowUsers 为否的时候运算 v-for。
反例
```
<ul>
  <li
    v-for="user in users"
    v-if="user.isActive"
    :key="user.id"
  >
    {{ user.name }}
  </li>
</ul>
<ul>
  <li
    v-for="user in users"
    v-if="shouldShowUsers"
    :key="user.id"
  >
    {{ user.name }}
  </li>
</ul>
```
好例子
```
<ul>
  <li
    v-for="user in activeUsers"
    :key="user.id"
  >
    {{ user.name }}
  </li>
</ul>
<ul v-if="shouldShowUsers">
  <li
    v-for="user in users"
    :key="user.id"
  >
    {{ user.name }}
  </li>
</ul>
```
## 为组件样式设置作用域
对于应用来说，顶级 App 组件和布局组件中的样式可以是全局的，但是其它所有组件都应该是有作用域的。

这条规则只和单文件组件有关。你不一定要使用 scoped 特性。设置作用域也可以通过 CSS Modules，那是一个基于 class 的类似 BEM 的策略，当然你也可以使用其它的库或约定。

不管怎样，对于组件库，我们应该更倾向于选用基于 class 的策略而不是 scoped 特性。

这让覆写内部样式更容易：使用了常人可理解的 class 名称且没有太高的选择器优先级，而且不太会导致冲突。

 详解
如果你和其他开发者一起开发一个大型工程，或有时引入三方 HTML/CSS (比如来自 Auth0)，设置一致的作用域会确保你的样式只会运用在它们想要作用的组件上。

不止要使用 scoped 特性，使用唯一的 class 名可以帮你确保那些三方库的 CSS 不会运用在你自己的 HTML 上。比如许多工程都使用了 button、btn 或 icon class 名，所以即便你不使用类似 BEM 的策略，添加一个 app 专属或组件专属的前缀 (比如 ButtonClose-icon) 也可以提供很多保护。

```
反例
<template>
  <button class="btn btn-close">X</button>
</template>

<style>
.btn-close {
  background-color: red;
}
</style>
好例子
<template>
  <button class="button button-close">X</button>
</template>

<!-- 使用 `scoped` 特性 -->
<style scoped>
.button {
  border: none;
  border-radius: 2px;
}

.button-close {
  background-color: red;
}
</style>
<template>
  <button :class="[$style.button, $style.buttonClose]">X</button>
</template>

<!-- 使用 CSS Modules -->
<style module>
.button {
  border: none;
  border-radius: 2px;
}

.buttonClose {
  background-color: red;
}
</style>
<template>
  <button class="c-Button c-Button--close">X</button>
</template>

<!-- 使用 BEM 约定 -->
<style>
.c-Button {
  border: none;
  border-radius: 2px;
}

.c-Button--close {
  background-color: red;
}
</style>
```

## 私有属性名 必要
在插件、混入等扩展中始终为自定义的私有属性使用 $_ 前缀。并附带一个命名空间以回避和其它作者的冲突 (比如 $_yourPluginName_)。
- 详解 
反例
```
var myGreatMixin = {
  // ...
  methods: {
    update: function () {
      // ...
    }
  }
}
var myGreatMixin = {
  // ...
  methods: {
    _update: function () {
      // ...
    }
  }
}
var myGreatMixin = {
  // ...
  methods: {
    $update: function () {
      // ...
    }
  }
}
var myGreatMixin = {
  // ...
  methods: {
    $_update: function () {
      // ...
    }
  }
}
```
好例子
```
var myGreatMixin = {
  // ...
  methods: {
    $_myGreatMixin_update: function () {
      // ...
    }
  }
}
```
## 单文件组件文件的大小写 强烈推荐
单文件组件的文件名应该要么始终是单词大写开头 (PascalCase)，要么始终是横线连接 (kebab-case)。

单词大写开头对于代码编辑器的自动补全最为友好，因为这使得我们在 JS(X) 和模板中引用组件的方式尽可能的一致。然而，混用文件命名方式有的时候会导致大小写不敏感的文件系统的问题，这也是横线连接命名同样完全可取的原因。

反例
```
components/
|- mycomponent.vue
components/
|- myComponent.vue
```
好例子
```
components/
|- MyComponent.vue
components/
|- my-component.vue
```

## 基础组件名 强烈推荐
应用特定样式和约定的基础组件 (也就是展示类的、无逻辑的或无状态的组件) 应该全部以一个特定的前缀开头，比如 Base、App 或 V。
- 详解 
反例
```
components/
|- MyButton.vue
|- VueTable.vue
|- Icon.vue
```
好例子
```
components/
|- BaseButton.vue
|- BaseTable.vue
|- BaseIcon.vue
components/
|- AppButton.vue
|- AppTable.vue
|- AppIcon.vue
components/
|- VButton.vue
|- VTable.vue
|- VIcon.vue
```

## 组件名中的单词顺序 强烈推荐
组件名应该以高级别的 (通常是一般化描述的) 单词开头，以描述性的修饰词结尾。

- 详解 
```
反例
components/
|- ClearSearchButton.vue
|- ExcludeFromSearchInput.vue
|- LaunchOnStartupCheckbox.vue
|- RunSearchButton.vue
|- SearchInput.vue
|- TermsCheckbox.vue
```
好例子
```
components/
|- SearchButtonClear.vue
|- SearchButtonRun.vue
|- SearchInputQuery.vue
|- SearchInputExcludeGlob.vue
|- SettingsCheckboxTerms.vue
|- SettingsCheckboxLaunchOnStartup.vue
```
## 简单的计算属性 强烈推荐
应该把复杂计算属性分割为尽可能多的更简单的属性。
- 详解 
反例
```
computed: {
  price: function () {
    var basePrice = this.manufactureCost / (1 - this.profitMargin)
    return (
      basePrice -
      basePrice * (this.discountPercent || 0)
    )
  }
}
```

好例子
```
computed: {
  basePrice: function () {
    return this.manufactureCost / (1 - this.profitMargin)
  },
  discount: function () {
    return this.basePrice * (this.discountPercent || 0)
  },
  finalPrice: function () {
    return this.basePrice - this.discount
  }
}
```



[查看详细规范](https://cn.vuejs.org/v2/style-guide/#%E8%A7%84%E5%88%99%E5%BD%92%E7%B1%BB)