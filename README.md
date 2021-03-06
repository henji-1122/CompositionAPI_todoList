#### 1、Vue 3.0 性能提升主要是通过哪几方面体现的？
* 解析：Vue 3.0 性能提升主要通过以下3个方面：
* 响应式系统升级
  - Vue3.0中使用Proxy(ES6新增)对象重写响应式系统，提升了性能和功能：
    * 可以监听动态新增的属性
    * 可以监听删除的属性
    * 可以监听数组的索引和length属性
  - Proxy本身的性能就比defineProperty要好，另外代理对象可以拦截属性的访问、赋值、删除等操作，不需要初始化的时候遍历所有的属性，多个属性嵌套的话，只有访问某个属性的时候才会递归处理下一级的属性
  - vue2中想要动态监听属性的变化需要使用.set方法进行处理，而且监听不到属性的删除，对数组的索引和length属性的也监听不到
* 编译优化
  - 通过优化编译的过程和重写DOM，让首次渲染和更新的功能有了大幅度的提升
* 源码体积的优化
  - vue.js 3.0中移除了一些不常用的API
    * 例如：inline-template、filter等(filter可通过method或者computed实现)
  - Tree-shaking
    * Tree-shaking依赖esmodule也就是ES6的模块化语法的静态结构，import和export，通过编译阶段的分析找到没有引入的模块在打包的时候直接过滤掉，让打包后的体积更小
    * 内置的组件如transition、keeplive、v-model都是按需引入的，vue3中的很多API都支持Tree-shaking，所以vue3中一些新增的API如果你没有使用这部分代码就不会被打包，只会打包你使用的API，但是默认vue的核心模块都会被打包。


#### 2、Vue 3.0 所采用的 Composition Api 与 Vue 2.x使用的Options Api 有什么区别？
* 解析：
  - Options Api是包含一个描述组件选项(data、methods、props等)的对象，采用这种方式开发组件时会创建data、methods、prop、生命周期钩子等，这些都是选项，这些选项组成了一个对象来描述组件，组件中不同的功能这些选项可能被拆分到不同的选项中，看他人开发的组件时会很难看懂，另外还难以提取组件中可重用的逻辑，虽然vue2中有minix混入的机制，可以把组件中重复的代码提取并重用，但是minix使用的过程会有问题(如命名冲突、数据来源不清晰)
  - Composition API提供了一组基于函数的API，更灵活的组织组件的逻辑，更合理的组织组件内部的代码结构，还可以把一些逻辑功能从组件中提取出来，方便其他组件重用。


#### 3、Proxy 相对于 Object.defineProperty 有哪些优点？
* 解析
  - vue2.x中响应式系统的核心defineProperty，在初始化的时候遍历data中的所有成员，通过defineProperty将对象的属性转换成getter和setter，如果data的属性中的值又是对象的话需要递归处理每一个子对象的属性，这些都是在初始化的时候进行的，如果没有使用某个属性，也会把她进行了响应式的处理。
  - vue2.x中想要动态监听属性的变化需要使用.set方法进行处理，而且监听不到属性的删除，对数组的索引和length属性的也监听不到。
  - Vue3.0中使用Proxy(ES6新增)对象重写响应式系统，Proxy本身的性能就比defineProperty要好，另外代理对象可以拦截属性的访问、赋值、删除等操作，不需要初始化的时候遍历所有的属性，多个属性嵌套的话，只有访问某个属性的时候才会递归处理下一级的属性。


#### 4、Vue 3.0 在编译方面有哪些优化？
* 解析
  - vue.js 3.0中标记和提升所有的静态根节点，diff的时候只需要对比动态节点内容
    * Fragments(升级vetur插件)：vue3中新增的片段特性，模板中不需要再创建一个唯一的根节点，模板中可以直接放文本内容或者很多同级的标签，在vscode中需要升级vetur插件，否则模板中没有唯一的根节点vscode中会报错
    * 静态提升
    * Patch flag
    * 缓存事件处理函数

#### 5、Vue.js 3.0 响应式系统的实现原理？
* 解析
    - Proxy对象实现属性监听，不需要初始化的时候就遍历所有的属性将其转换为响应式数据
    - 多层属性嵌套，只有在访问属性过程中处理下一级属性
    - 默认监听动态添加的属性
    - 默认监听属性的删除操作
    - 默认监听数组索引和length属性
    - 可以作为单独的模块使用