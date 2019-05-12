## vue开发规范

### 组件数据
> 组件的 data 必须是一个函数。
> 
> 当在组件中使用 data 属性的时候 (除了 new Vue 外的任何地方)，它的值必须是返回一个对象的函数。

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
### Prop 定义
> Prop 定义应该尽量详细。
> 
> 在你提交的代码中，prop 的定义应该尽量详细，至少需要指定其类型。
反例
```
// 这样做只有开发原型系统时可以接受
props: ['status']
```
好例子
```
props: {
  status: String
}
```
更好的做法！
```
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
### 为 v-for 设置键值

> 总是用 key 配合 v-for。
> 
> 在组件上总是必须用 key 配合 v-for，以便维护内部组件及其子树的状态。甚至在元素上维护可预测的行为，比如动画中的对象固化 (object constancy)，也是一种好的做法。

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
### 避免 v-if 和 v-for 用在一起

> 永远不要把 v-if 和 v-for 同时用在同一个元素上。
> 
> 一般我们在两种常见的情况下会倾向于这样做：
> 
> + 为了过滤一个列表中的项目 (比如 v-for="user in users" v-if="user.isActive")。在这种情形下，请将 users 替换为一个计算属性 (比如 activeUsers)，让其返回过滤后的列表。
> 
> + 为了避免渲染本应该被隐藏的列表 (比如 v-for="user in users" v-if="shouldShowUsers")。这种情形下，请将 v-if 移动至容器元素上 (比如 ul, ol)。

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
### 为组件样式设置作用域

> 对于应用来说，顶级 App 组件和布局组件中的样式可以是全局的，但是其它所有组件都应该是有作用域的。
> 
> 这条规则只和单文件组件有关。你不一定要使用 scoped 特性。设置作用域也可以通过 CSS Modules，那是一个基于 class 的类似 BEM 的策略，当然你也可以使用其它的库或约定。
> 
> 不管怎样，对于组件库，我们应该更倾向于选用基于 class 的策略而不是 scoped 特性。
> 
> 这让覆写内部样式更容易：使用了常人可理解的 class 名称且没有太高的选择器优先级，而且不太会导致冲突。

反例
```
<template>
  <button class="btn btn-close">X</button>
</template>

<style>
.btn-close {
  background-color: red;
}
</style>
```
好例子
```
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
### 组件文件

> 只要有能够拼接文件的构建系统，就把每个组件单独分成文件。
> 
> 当你需要编辑一个组件或查阅一个组件的用法时，可以更快速的找到它。

反例
```
Vue.component('TodoList', {
  // ...
})

Vue.component('TodoItem', {
  // ...
})
```
好例子
```
components/
|- TodoList.js
|- TodoItem.js

components/
|- TodoList.vue
|- TodoItem.vue
```
### 单文件组件文件的大小写

> 单文件组件的文件名应该始终是单词大写开头 (PascalCase)
> 
> 单词大写开头对于代码编辑器的自动补全最为友好，因为这使得我们在 JS(X) 和模板中引用组件的方式尽可能的一致。

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
```
### 基础组件名

> 应用特定样式和约定的基础组件 (也就是展示类的、无逻辑的或无状态的组件) 应该全部以一个特定的前缀开头，比如 Base、App 或 V。

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
### 紧密耦合的组件名

> 和父组件紧密耦合的子组件应该以父组件名作为前缀命名。
> 
> 如果一个组件只在某个父组件的场景下有意义，这层关系应该体现在其名字上。因为编辑器通常会按字母顺序组织文件，所以这样做可以把相关联的文件排在一起。

反例
```
components/
|- TodoList.vue
|- TodoItem.vue
|- TodoButton.vue

components/
|- SearchSidebar.vue
|- NavigationForSearchSidebar.vue
```
好例子
```
components/
|- TodoList.vue
|- TodoListItem.vue
|- TodoListItemButton.vue

components/
|- SearchSidebar.vue
|- SearchSidebarNavigation.vue
```
### 组件名中的单词顺序

> 组件名应该以高级别的(通常是一般化描述的) 单词开头，以描述性的修饰词结尾。

反例
```
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
### 自闭合组件

> 在单文件组件、字符串模板和 JSX 中没有内容的组件应该是自闭合的——但在 DOM 模板里永远不要这样做。
> 
> 自闭合组件表示它们不仅没有内容，而且刻意没有内容。其不同之处就好像书上的一页白纸对比贴有“本页有意留白”标签的白纸。而且没有了额外的闭合标签，你的代码也更简洁。
> 
> 不幸的是，HTML 并不支持自闭合的自定义元素——只有官方的“空”元素。所以上述策略仅适用于进入 DOM 之前 Vue 的模板编译器能够触达的地方，然后再产出符合 DOM 规范的 HTML。

反例
```
<!-- 在单文件组件、字符串模板和 JSX 中 -->
<my-component></my-component>

<!-- 在 DOM 模板中 -->
<my-component/>
```
好例子
```
<!-- 在单文件组件、字符串模板和 JSX 中 -->
<my-component/>

<!-- 在 DOM 模板中 -->
<my-component></my-component>
```
### 模板中的组件名大小写

> 组件模板中的组件名应该始终是kebab-case

反例
```
<!-- 在单文件组件和字符串模板中 -->
<mycomponent/>

<!-- 在单文件组件和字符串模板中 -->
<myComponent/>

<!-- 在 DOM 模板中 -->
<MyComponent></MyComponent>
```
好例子
```
<my-component></my-component>
```
### JS/JSX 中的组件名大小写
> 
> JS/JSX 中的组件名应该始终是 PascalCase 的，尽管在较为简单的应用中只使用 Vue.component 进行全局组件注册时，可以使用 kebab-case 字符串。

反例
```
Vue.component('myComponent', {
  // ...
})

import myComponent from './MyComponent.vue'

export default {
  name: 'myComponent',
  // ...
}

export default {
  name: 'my-component',
  // ...
}
```
好例子
```
Vue.component('MyComponent', {
  // ...
})

Vue.component('my-component', {
  // ...
})

import MyComponent from './MyComponent.vue'

export default {
  name: 'MyComponent',
  // ...
}
```
### Prop 名大小写

> 在声明 prop 的时候，其命名应该始终使用 camelCase，而在模板和 JSX 中应该始终使用 kebab-case。
> 
> 我们单纯的遵循每个语言的约定。在 JavaScript 中更自然的是 camelCase。而在 HTML 中则是 kebab-case。

反例
```
props: {
  'greeting-text': String
}

<welcome-message greetingText="hi"/>
```
好例子
```
props: {
  greetingText: String
}

<welcome-message greeting-text="hi"/>
```
### 多个特性的元素

> 多个特性的元素应该分多行撰写，每个特性一行。
> 
> 在 JavaScript 中，用多行分隔对象的多个属性是很常见的最佳实践，因为这样更易读。模板和 JSX 值得我们做相同的考虑。

反例
```
<img src="https://vuejs.org/images/logo.png" alt="Vue Logo">

<my-component foo="a" bar="b" baz="c"/>
```
好例子
```
<img
  src="https://vuejs.org/images/logo.png"
  alt="Vue Logo"
>

<my-component
  foo="a"
  bar="b"
  baz="c"
/>
```
### 模板中简单的表达式

> 组件模板应该只包含简单的表达式，复杂的表达式则应该重构为计算属性或方法。
> 
> 复杂表达式会让你的模板变得不那么声明式。我们应该尽量描述应该出现的是什么，而非如何计算那个值。而且计算属性和方法使得代码可以重用。

反例
{% raw %}
```
{{
  fullName.split(' ').map(word => {
    return word[0].toUpperCase() + word.slice(1)
  }).join(' ')
}}
```
{% endraw %}

好例子
```
<!-- 在模板中 -->
{{ normalizedFullName }}

// 复杂表达式已经移入一个计算属性
computed: {
  normalizedFullName: function () {
    return this.fullName.split(' ').map(function (word) {
      return word[0].toUpperCase() + word.slice(1)
    }).join(' ')
  }
}
```
### 简单的计算属性

> 应该把复杂计算属性分割为尽可能多的更简单的属性。

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
### 带引号的特性值

> 非空 HTML 特性值应该始终带双引号
> 
> 在 HTML 中不带空格的特性值是可以没有引号的，但这鼓励了大家在特征值里不写空格，导致可读性变差。

反例
```
<input type=text>

<app-sidebar :style={width:sidebarWidth+'px'}>
```
好例子
```
<input type="text">

<AppSidebar :style="{ width: sidebarWidth + 'px' }">
```
### 指令缩写

> 指令缩写 (用 : 表示 v-bind: 和用 @ 表示 v-on:)

反例
```
<input
  v-bind:value="newTodoText"
  :placeholder="newTodoInstructions"
>

<input
  v-on:input="onInput"
  @focus="onFocus"
>
```
好例子
```
<input
  :value="newTodoText"
  :placeholder="newTodoInstructions"
>

<input
  @input="onInput"
  @focus="onFocus"
>
```
### 单文件组件的顶级元素的顺序

> 单文件组件应该总是让 ` <script> `、` <template> ` 和 ` <style> ` 标签的顺序保持一致。且 ` <style> ` 要放在最后，因为另外两个标签至少要有一个。

反例
```
<style>/* ... */</style>
<script>/* ... */</script>
<template>...</template>

<!-- ComponentA.vue -->
<script>/* ... */</script>
<template>...</template>
<style>/* ... */</style>
```
好例子
```
<!-- ComponentA.vue -->
<template>...</template>
<script>/* ... */</script>
<style>/* ... */</style>

<!-- ComponentB.vue -->
<template>...</template>
<script>/* ... */</script>
<style>/* ... */</style>
```
### scoped 中的元素选择器

> 元素选择器应该避免在 scoped 中出现。
> 
> 在 scoped 样式中，类选择器比元素选择器更好，因为大量使用元素选择器是很慢的。

反例
```
<template>
  <button>X</button>
</template>

<style scoped>
button {
  background-color: red;
}
</style>
```
好例子
```
<template>
  <button class="btn btn-close">X</button>
</template>

<style scoped>
.btn-close {
  background-color: red;
}
</style>
```
### 隐性的父子组件通信

> 应该优先通过 prop 和事件进行父子组件之间的通信，而不是 this.$parent 或改变 prop。
> 
> 一个理想的 Vue 应用是 prop 向下传递，事件向上传递的。遵循这一约定会让你的组件更易于理解。然而，在一些边界情况下 prop 的变更或 this.$parent 能够简化两个深度耦合的组件。
> 
> 问题在于，这种做法在很多简单的场景下可能会更方便。但请当心，不要为了一时方便 (少写代码) 而牺牲数据流向的简洁性 (易于理解)。

反例
```
Vue.component('TodoItem', {
  props: {
    todo: {
      type: Object,
      required: true
    }
  },
  template: '<input v-model="todo.text">'
})

Vue.component('TodoItem', {
  props: {
    todo: {
      type: Object,
      required: true
    }
  },
  methods: {
    removeTodo () {
      let vm = this
      vm.$parent.todos = vm.$parent.todos.filter(function (todo) {
        return todo.id !== vm.todo.id
      })
    }
  },
  template: `
    <span>
      {{ todo.text }}
      <button @click="removeTodo">
        X
      </button>
    </span>
  `
})
```
好例子
```
Vue.component('TodoItem', {
  props: {
    todo: {
      type: Object,
      required: true
    }
  },
  template: `
    <input
      :value="todo.text"
      @input="$emit('input', $event.target.value)"
    >
  `
})

Vue.component('TodoItem', {
  props: {
    todo: {
      type: Object,
      required: true
    }
  },
  template: `
    <span>
      {{ todo.text }}
      <button @click="$emit('delete')">
        X
      </button>
    </span>
  `
})
```
### 禁止在计算属性中出现异步操作
> 计算属性应该是同步的. 计算属性中的异步操作不能按预期地执行,并且可能引发异常

反例
```
<script>
export default {
  computed: {
    pro () {
      return Promise.all([new Promise((resolve, reject) => {})])
    },
    foo1: async function () {
      return await someFunc()
    },
    bar () {
      return fetch(url).then(response => {})
    },
    tim () {
      setTimeout(() => { }, 0)
    },
    inter () {
      setInterval(() => { }, 0)
    },
    anim () {
      requestAnimationFrame(() => {})
    }
  }
}
</script>
```
好例子
```
export default {
  computed: {
    foo () {
      let bar = 0
      try {
        bar = bar / this.a
      } catch (e) {
        return 0
      } finally {
        return bar
      }
    }
  }
}
</script>
```
### 不要写有副作用的计算属性
> 在计算属性中引入副作用被认为是一种非常糟糕的做法,它使代码不可预测且难以理解.

反例
```
<script>
export default {
  computed: {
    fullName () {
      this.firstName = 'lorem'
      return `${this.firstName} ${this.lastName}` // 引发副作用,this.firstName被改变
    },
    reversedArray () {
      return this.array.reverse() // 引发副作用,原数组被改变了
    }
  }
}
</script>

```
好例子
```
<script>
export default {
  computed: {
    fullName () {
      return `${this.firstName} ${this.lastName}`
    },
    reversedArray () {
      return this.array.slice(0).reverse() // .slice(0)产生原数组的副本,原数组没被改变
    }
  }
}
</script>
```
### 计算属性必需有返回值

反例
```
<script>
export default {
  computed: {
    baz () {
      if (this.baf) {
        return this.baf
      }
    },
    baf: function () {}
  }
}
</script>

```
好例子
```
<script>
export default {
  computed: {
    foo () {
      if (this.bar) {
        return this.baz
      } else {
        return this.baf
      }
    },
    bar: function () {
      return false
    }
  }
}
</script>
```
### props的默认值应该是有效的

反例
```
script>
export default {
  props: {
    propA: {
      type: String,
      default: {}
    },
    propB: {
      type: String,
      default: []
    },
    propC: {
      type: Object,
      default: []
    },
    propD: {
      type: Array,
      default: []
    },
    propE: {
      type: Object,
      default: { message: 'hello' }
    }
  }
}
</script>
```
好例子
```
script>
export default {
  props: {
    propA: Number,
    propB: [String, Number],
    propD: {
      type: Number,
      default: 100
    },
    // Array和Object类型的default值应该是一个返回对应类型值的函数
    propE: {
      type: Object,
      default() {
        return { message: 'hello' }
      }
    },
    propF: {
      type: Array,
      default() {
        return []
      }
    }
  }
}
</script>
```

