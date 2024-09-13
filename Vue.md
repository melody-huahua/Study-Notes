

### Vue3 常用 Composition API

#### 1. setup函数

 1. 理解: Vue3中一个新的配置项, 值为一个函数

 2. 组件中所用到的数据、方法等, 均要配置在setup中

 3. setup函数的两种返回值: 

    (1) 若返回一个对象, 则对象中的属性、方法, 在模版中居客直接使用

    (2) 若返回一个渲染函数, 则可以自定义渲染内容

 4. 不要与Vue2混用, Vue2可以访问setup中的属性方法, 但setup不能访问Vue2 ; 如果出现重名, setup优先

 5. setup不能是一个async函数, 因为返回值不在是return的对象,而是promise

#### 2. ref函数

1. 作用: 定义一个响应式数据, 创建一个包含响应式数据的引用对象(reference对象)
2. 可以接收基本类型, 也可以是对象类型; 基本类型响应式依靠defineProperty()的get、set完成; 对象类型依靠Vue3内部的reactive()

#### 3. reactive函数

1. 作用: 定义一个对象类型的响应式数据(基本类型不可用)
1. 接收一个源对象(对象或数组), 返回一个代理对象(Proxy的实例对象)
1. reactive定义的响应式数据是 “深层次的”
1. 基于ES6的proxy实现, 通过 代理对象 操作 源对象

#### 4. ??响应式原理

1. Vue2中defineProperty中存在的问题:

   新增属性, 删除属性, 界面不会更新; 直接通过下标修改数组, 界面不会更新; 因为set和get无法捕获未定义的属性

2. Vue3的响应式原理: 

   (1) 通过Proxy(代理): 拦截对象中任意属性的变化, 包括: 属性值的读写、 属性的添加、 属性的删除等

   (2) 通过Reflect(反射): 对被代理对象的属性进行操作

   (3) 比较土的解释:  new Proxy ( 属性 , { 配置项 })  , Proxy配置项中比defineProperty多了deleteProperty; 且set可以捕获追加, 解决了Vue2的问题

   (4) ??原理探索( 还没看懂 ):   ??window.Reflect(反射对象)与Proxy的配合

#### 5. reactive对比ref

* 从定义数据角度对比: 

  (1) ref用来定义基本类型数据

  (2) reactive用来定义对象(或数组)类型数据

  (3) ref也可以定义对象(或数组)类型, 他的内部会通过reactive代理

* 从原理角度对比: 

  (1) ref通过 defineProperty 的get和set数据劫持实现响应式

  (2) reactive通过 Proxy 实现数据劫持, 并通过 Reflect 操作源对象内部的数据

* 从实用角度对比

  (1) ref定义的数据: 操作数据需要.value, 读取数据模版中不需要.value

  (2) reactive定义的数据, 均不需要.value

#### 6. setup的两个注意点 

1. setup() 比 beforeCreate 要早, 此时this没有

2. Setup能收到两个参数

   (1) 第一个参数接收父组件传的值

   (2)第二个参数是上下文对象(包含attrs、slots、emit)

#### 7. 计算属性和watch

   1. Computed

      (1) 完整写法(考虑读和写)

      ```javascript
      let fullName = computed({
      	get(){ return XX }
      	set(value){ dosomething...  }
      })
      ```

   2. Watch监听

      (1) 基本用法
      
      ```js
        watch(
            [sum, msg],
            (newV, oldV) => {
              console.log(newV, oldV);
            },
            { immediate: true, deep: true }
          );
      ```
      
      (2) 踩坑
      
      	1. reactive监听无法获取oldValue
      	1. 监听reactive时, 若要监听对象中的某一属性, 需要将目标写为函数形式
      
      (3)  watchEffect
      
      ```
      watchEffect(()=>{
      	// 用到啥监听啥
      })
      ```
      

#### 8. 生命周期

#### 9. hook

- 什么是hook?  ——本质是一个函数, 把setup函数中使用的Composition API进行了封装
- 类似于vue2.x中的mixin
- 自定义hook的优势: 复用代码, 让setup中的逻辑更清楚易懂

#### 10. toRef 和 toRefs

- 作用: 创建一个ref对象, 其value值指向另一个对象中的某个属性
- 语法: const name = toRef ( person ,' name' )   /  toRefs ( person ) 
- 应用: 要将响应式对象中的某个属性单独提供给外部使用时

### 其他 Composition API

#### 1.  shallowReacitve和shallowRef

- shallowReacitve : 只处理对象最外层属性的响应式( 浅响应式 )

- shallowRef: 只处理基本数据类型的响应式, 不进行对象的响应式处理

- 什么时候使用? 

  如果有一个对象数据, 结构比较深, 但变化时只有外层属性变化 => shallowReactive
  
  如果有一个对象数据, 后续功能不会修改该对象中的属性, 而是生成新的对象来替换 => shallowRef
  
#### 2.  readonly 和 shallowReadonly

- randonly : 让一个响应式数据变为只读的 (深只读)
- shallowReadonly : 让一个响应式数据变为只读的 (浅只读)





### Pinia

#### 准备工作

```javascript
// 安装
npm i pinia

// main引入
import { createPinia } from 'pinia'
const pinia = createPinia();
createApp(App).use(pinia).mount('#app')
```

#### 读写数据

```javascript
// 1. 新建文件夹Store和仓库Count.ts
import { defineStore } from 'pinia'

export const useCountStore = defineStore('count', {
  // 修改数据的方法（需要带逻辑的修改, 比如请求接口）
   actions: {
        async increment(value) {
            if (this.sum < 10) {
                this.sum += value
            }

        }
    },
  
  
    state() {
        return {
            sum: 1,
        }
    }
})


// 2. 页面中使用
import { useCounterStore } from '@/store/XXXX'
const CountStore = useCountStore();

// 修改数据的方法（直接修改）
CountStore.sum++

// 修改数据的方法(批量修改)
CountStore.$patch({
  sum:888,
  address:'哈哈'
})

// 修改数据的方法（需要带逻辑的修改）
CountStore.increment(3)
```

#### getters

```javascript
  // 两种方式拿到state       (1) 直接this    (2) 传参
  getters: {
        big(): number {
            console.log(this, 'this');
            return this.sum * 2
        },
        big2: state => {
            return state.sum * 3
        }
    }
```



#### 小技巧

```javascript
// 1. 解构后失去响应式使用 storeToRefs 进行处理
const { count, double } = storeToRefs(counter)

//--------------------------------------------------------------------------------

// 2. $subscribe: 订阅 ; 下面是一个本地存储的示例
// vue模块中: 
CountStore.$subscribe((mutate, state) => {
  localStorage.setItem("list", JSON.stringify(state.list));
});	
// CountStore的state中
 state() {
        return {
            list: JSON.parse(localStorage.getItem('list') as string) || []  // 断言+边缘处理
        }
    },
      
//--------------------------------------------------------------------------------
      
// 3. 组合式写法
import { defineStore } from 'pinia'
import { ref } from 'vue'
export const useCounterStore = defineStore('count', () => {
    // 数据（state）
    const count = ref(0)
    // 修改数据的方法（action）
    const increment = () => {
        count.value++
    }
    // 已对象的形式返回
    return {
        count,
        increment
    }
})      
```

### 组件通信

#### 1. props

```javascript
// 父---
<script setup lang="ts">
import son from "./components/son.vue";
import { ref } from "vue";
const a = 1;
function sendB(params: number) {
  console.log(params, "子过来的value");
}
</script>
<template>
  <son :a="a" :sendB="sendB" />
</template>

// 子---
<script setup lang="ts">
defineProps(["a", "sendB"]);
</script>
<template>
  <p>{{ a }}</p>
  <button @click="sendB(3)">给父发送一个3</button>
</template>

```

#### 2. 自定义事件emit

```javascript
// 子: 自定义事件
 <button @click="sendValue">给父亲发送一个value</button>

const emit = defineEmits(["send-value"]);
function sendValue() {
  emit("send-value", "子组件发送的一个假数据value1111");
}

// 父: 接收
<template>
  <son @send-value="savaValue" />
</template>
	
function savaValue(params) {
  console.log(params, "过来了");
}
```

#### 3. mitt 消息订阅与发布

```javascript
// 安装
npm i mitt

// 新建文件 src\utils\emitter.ts
// 引入mitt 
import mitt from "mitt";
// 创建emitter
const emitter = mitt()
// 创建并暴露mitt
export default emitter
```

注册

```javascript
import emitter from "@/utils/emitter";
import { onUnmounted } from "vue";

// 绑定事件
emitter.on('send-toy',(value)=>{
  console.log('send-toy事件被触发',value)
})

onUnmounted(()=>{
  // 解绑事件
  emitter.off('send-toy')
})
```

使用

```javascript
import emitter from "@/utils/emitter";

function sendToy(){
  // 触发事件
  emitter.emit('send-toy',toy.value)
}
```

#### 4. v-model

1. v-mode本质

```vue
<!-- 使用v-model指令 -->
<input type="text" v-model="userName">

<!-- v-model的本质是下面这行代码 -->
<input 
  type="text" 
  :value="userName" 
  @input="userName =(<HTMLInputElement>$event.target).value"
>
```

2. 组件上 v-model 本质, 相当于在组件上: `:moldeValue` ＋ `update:modelValue`事件。

```vue
<!-- 组件标签上使用v-model指令 -->
<AtguiguInput v-model="userName"/>

<!-- 组件标签上v-model的本质 -->
<AtguiguInput :modelValue="userName" @update:model-value="userName = $event"/>
```



```vue
// 组件中
<template>
  <div class="box">
    <!--将接收的value值赋给input元素的value属性，目的是：为了呈现数据 -->
		<!--给input元素绑定原生input事件，触发input事件时，进而触发update:model-value事件-->
    <input 
       type="text" 
       :value="modelValue" 
       @input="emit('update:model-value',$event.target.value)"
    >
  </div>
</template>

<script setup lang="ts" name="AtguiguInput">
  // 接收props
  defineProps(['modelValue'])
  // 声明事件
  const emit = defineEmits(['update:model-value'])
</script>
```

#### 5. $refs、$parent

```

```



### Vue2补充

#### 1. 关于脚手架配置

- vue查看默认webpack配置 : vue inspect > output.js
- 更改默认配置 : vue.config.js , 文档: https://cli.vuejs.org/zh/

#### 2. mixin混入

#### 3. 过滤器

#### 4. 自定义指令

#### 5. 插件

- 作用: 用于增强Vue

- 本质: 包含install方法的一个对象, install的第一个参数是Vue, 第二个以后的参数是插件使用者传递的数据

- 定义插件:

  ```
  对象.install = function (Vue, options){
  	// 全局过滤器
  	Vue.filter()
  	
  	// 全局指令
  	Vue.directive()
  	
  	// 配置全局混入
  	Vue.mixin()
  	
  	// 添加实例方法
  	Vue.prototype.$myMethod = function(){}
  }
  ```
- 使用: Vue.use()





























