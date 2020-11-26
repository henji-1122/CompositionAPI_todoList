#### Composition API

* createApp函数：作用是创建一个vue对象，接收一个选项作为参数，也就是组件的选项，返回vue对象

* setup函数：是composition API的入口
  - setup()返回一个对象，返回的对象可使用在模板、methods、computed以及生命周期钩子函数中
  - setup()执行的时机：是在props被解析完毕，组件被创建之前执行的，所以在setup()中无法通过this获取到组件的实例，因为实例还未被创建，所以在setup()中无法访问到组件中的data、methods、computed，setup()内部的this指向的是undefined

* reactive()：创建响应式对象，把对象转换成响应式的对象，是一个代理对象
  - 之前是在data中设置响应式的数据，这里还可以这么做，但是为了让某一个逻辑的所有代码都封装在一个函数中，vue3中新增了一个API,reactive用来创建响应式对象，并且这个对象的嵌套属性也会是响应式的，返回的是一个proxy对象

* toRefs()：将代理对象中的所有属性都转换成响应式对象
  - 传入的对象必须是一个代理对象
  - 他内部会创建一个新的对象，然后遍历这个传入代理对象中的所有属性，将所有属性都转换成响应式对象，然后挂载到新的对象上，最后把新创建的这个对象返回，他内部会对代理对象的每个属性创建一个具有value属性的对象，该对象是响应式的，value属性具有getter和setter，getter中返回代理对象中对应属性的值，setter中给代理对象的属性赋值，所以返回的每一个属性都是响应式的
  - 所以可以解构toRefs返回的对象，解构的每一个属性也都是响应式的，每个属性是一个具有value属性的对象
  - toRefs在处理这个对象属性时，类似于ref()，通过toRefs()处理reactive()返回的代理对象可以进行解构的操作

* ref()：将一个基本类型数据转换成响应式对象
  - 和reactive()不同的是，reactive()是将对象转换成响应式的数据，ref()是把基本类型的数据包装成响应式的对象
  - const count = ref(0)将count包装成响应式的对象，初始值为0，调用ref返回的count是一个对象，他只有一个value属性，value的值就是0，并且value属性是有getter和setter的，因为要拦截这个数据的访问和赋值,count在模板中使用时省略value
  - 在使用count进行解构，解构出的count也是响应式的对象，解构出的count和定义的count指向内存中的同一个地址

* computed：简化模板中的代码，缓存计算的结果，当数据变化后才会重新计算
  - 在使用computed任然可以采用2.x中在创建组件的时候传入computed选项来创建计算属性，在vue3中在setup()中使用computed函数来创建计算属性

* Watch
  - 跟computed类似，可以在setup()中创建侦听器
  - 使用方式跟之前使用$watch()或者选项中的watch是一样的，监听响应式数据的变化，然后执行一个相应的回调函数，可以监听数据的新值和旧值
  - Watch的三个参数
    - 第一个参数：要监听的数据(获取值的函数，监听这个函数返回值的变化 | ref() | reactive()返回的对象 | 数组)
    - 第二个参数：监听到数据变化后执行的函数，这个函数有两个参数分别是新值和旧值
    - 第三个参数：选项对象，deep深度监听和immediate立即执行
  - Watch的返回值(一个函数)
    - 取消监听的函数
  - setup()中watch使用跟过去的this.$watch是一样的，不一样的是第一个参数不是字符串，而是ref()或者是reactive()返回的对象

* watchEffect
  - 是watch函数的简化版本，也用来监视数据的变化
    - 内部实现和watch调用同一个函数doWatch，不同的是watchEffect没有第二个回调函数的参数
  - 接收一个函数作为参数，监听函数内响应式数据的变化，会立即执行一次这个函数
    - 当数据变化会重新执行该函数
    - 还返回一个取消监听的函数

* Demo: todoList 
  * 安装
    - npm install
  * 运行
    - npm run serve
