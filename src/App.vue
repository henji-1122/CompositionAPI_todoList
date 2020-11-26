<template>
  <section id="app" class="todoapp">
    <header class="header">
      <h1>todos</h1>
      <input
        class="new-todo"
        placeholder="What needs to be done?"
        autocomplete="off"
        autofocus
        v-model="input"
        @keyup.enter="addTodo"
        >
    </header>
    <section class="main" v-show="count">
      <input id="toggle-all" class="toggle-all" type="checkbox" v-model="allDone"> <!-- 所有待办项的状态 -->
      <label for="toggle-all">Mark all as complete</label>
      <ul class="todo-list">
        <li v-for="todo in filteredTodos" :key="todo"
        :class="{ editing : todo === editingTodo, completed: todo.completed}">
          <div class="view">
            <input class="toggle" type="checkbox" v-model="todo.completed">
            <label @dblclick="editTodo(todo)">{{ todo.text }}</label>
            <button class="destroy" @click="remove(todo)"></button>
          </div>
          <!-- 编辑的文本框 -->
          <input
            class="edit"
            type="text"
            v-editing-focus="todo === editingTodo"
            v-model="todo.text"
            @keyup.enter="doneEdit(todo)"
            @blur="doneEdit(todo)"
            @keyup.esc="cancelEdit(todo)"
            >
        </li>
      </ul>
    </section>
    <footer class="footer" v-show="count">
      <span class="todo-count">
        <strong>{{ remainingCount }}</strong>{{ remainingCount > 1 ? 'items' : 'item'}} left <!-- 未完成的待办项个数 -->
      </span>
      <ul class="filters">
        <li><a href="#/all">All</a></li>
        <li><a href="#/active">Active</a></li>
        <li><a href="#/completed">Completed</a></li>
      </ul>
      <button class="clear-completed" @click="removeCompleted" v-show="count > remainingCount"> <!-- 删除已完成的项 -->
        Clear completed
      </button>
    </footer>
  </section>
  <footer class="info">
    <p>Double-click to edit a todo</p>
    <!-- Remove the below line ↓ -->
    <p>Template by <a href="http://sindresorhus.com">Sindre Sorhus</a></p>
    <!-- Change this out with your name and url ↓ -->
    <p>Created by <a href="https://github.com/henji-1122">虫虫</a></p>
    <p>Part of <a href="http://todomvc.com">TodoMVC</a></p>
  </footer>
</template>

<script>
import './assets/index.css'
import useLocalStorage from './utils/useLocalStorage.js'
import { computed, onMounted, onUnmounted, ref, watchEffect } from 'vue'

const storage = useLocalStorage()

// 1. 添加待办事项
const useAdd = todos => {
  const input = ref('')
  const addTodo = () => {
    const text = input.value && input.value.trim()
    if(text.length === 0) return
    todos.value.unshift({
      text,
      completed: false
    })
    input.value = ''
  }
  return {
    input,
    addTodo
  }
}

// 2. 删除待办事项
const useRemove = todos => {
  const remove = todo => {
    const index = todos.value.indexOf(todo)
    todos.value.splice(index, 1)
  }

  // 删除已完成事项
  const removeCompleted = () => {
    todos.value = todos.value.filter(todo => !todo.completed)
  }

  return {
    remove,
    removeCompleted
  }
}

// 3. 编辑待办事项
const useEdit = remove => {
  let beforeEditingText = '' // 编辑前的文本
  const editingTodo = ref(null) // 编辑的项

  const editTodo = todo => { // 编辑函数
    beforeEditingText = todo.text // 记录编辑前的文本
    editingTodo.value = todo // 记录编辑项
  }
  const doneEdit = todo => { // 按回车或者文本框失去焦点时修改数据的函数
    if(!editingTodo.value) return // 如果不是编辑状态直接返回
    todo.text = todo.text.trim()
    todo.text || remove(todo) // 如果输入框没有值则直接删除此项
    editingTodo.value = null // 取消编辑状态
  }
  const cancelEdit = todo => { // 取消编辑的函数
    editingTodo.value = null
    todo.text = beforeEditingText // 将数据还原
  }

  return {
    editingTodo,
    editTodo,
    doneEdit,
    cancelEdit
  }
} 

// 4. 切换待办项完成状态
const useFilter = todos => {
  // 选中|未选中 所有
  const allDone = computed({
    get () {
      // 所有待办项完成返回true，否则返回false
      return !todos.value.filter(todo => !todo.completed).length
    },
    set (value) {
      todos.value.forEach(todo => {
        todo.completed = value
      })
    }
  })

  // 计算属性：待办项的总数
  const count = computed(() => todos.value.length)

  // 切换all active completed
  const filter = {
    all: list => list,
    active: list => list.filter(todo => !todo.completed), // 未完成待办
    completed: list => list.filter(todo => todo.completed) // 已完成待办
  }

  const type = ref('all')
  const filteredTodos = computed(() => filter[type.value](todos.value)) // 计算属性：执行filter中对应的函数返回不同状态的待办项
  const remainingCount = computed(() => filter.active(todos.value).length) // 计算属性：返回未完成的待办项个数
  const onHashchange = () => {
    const hash = window.location.hash.replace('#/', '')
    if (filter[hash]) { // 如果hash存在
      type.value = hash
    }else {
      type.value = 'all'
      window.location.hash =''
    }
  }

  onMounted (() => { // 监听hash改变事件
    window.addEventListener('hashchange', onHashchange)
    onHashchange()
  })

  onUnmounted (() => { // 取消监听hash改变事件
    window.removeEventListener('hashchange', onHashchange)
  })

  return {
    allDone,
    count,
    filteredTodos,
    remainingCount
  }
}
 
// 5. 存储待办事项
const useStorage = () => {
  const KEY = 'TODOKEYS' // 本地存储键
  const todos = ref(storage.getItem(KEY) || [])
  watchEffect(() => { //监视todos数据的变化，当数据发送变化会执行回调
    storage.setItem(KEY, todos.value)
  })
  return todos
}

export default {
  name: 'App',
  setup () {
    const todos = useStorage()

    const { remove, removeCompleted } = useRemove(todos)
    return {
      todos,
      remove,
      removeCompleted,
      ...useAdd(todos), //解构返回值
      ...useEdit(remove),
      ...useFilter(todos)
    }
  },

  // 自定义指令：获取焦点
  directives: {
    editingFocus: (el, binding) => { // el：指令修饰的元素，binding：获取到一些参数
      binding.value && el.focus() // 如果是编辑状态，则获取焦点
    }
  }
}
</script>

<style>
</style>
