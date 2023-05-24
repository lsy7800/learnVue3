# vue3.0

## 1. 初识vue3

create-vue是Vue官方新的脚手架工具，底层切换到了Vite

1. 前提环境条件

   已安装16.0或更高版本的Node.js

   ```cmdline
   node -v
   ```

   

2. 创建一个Vue应用

   ```cmdline
   npm init vue@latest
   ```

   ```cmdline
   npm create vite@latest
   ```

   推荐使用第一种方式创建项目

3. 认识目录

   * vite.config.js - 项目配置文件 <font color=red>基于vite的配置</font>
   * package.json - 项目包文件 核心以来项编程了vue3和vite
   * main.js - 入口文件 createApp函数创建应用实例
   * app.vue - 根组件 SFC单文件组件 script - template - style
     - 变化一： 脚本script 和模板template顺序进行了调整
     - 变化二：模板template不再要求唯一根元素
     - 变化三：脚本script添加setup标识支持组合式API
   * index.html - 单页面入口 提供id 为app挂载点

## 2. 组合式API

### 1.setup选项

写法

```vue
<script>
export default {
    setup(){
        console.log('setup')
        // 声明变量
        const message = 'this is a message'
        // 声明方法
        const logMessage = () => {
            console.log(message)
        }
        // 以对象的形式return
        return {
            message,
            logMessage
        }
    },
    // 生命周期函数
    beforeCreate(){
        
    }
}
</script>
```

执行时机

setup函数的执行是在beforeCreate之前执行的，且自动执行

setup 语法糖

使用语法糖简化上述写法

```vue
<script setup>   
    // 数据
    const message = 'this is a message'
    // 函数
    const logMessage = () => {
        console.log(message)
    }
</script>
```

* 注意：setup中的this不再指向组件实例

### 2.reactive和ref函数

reactive()

作用：接受对象类型数据的参数传入并返回一个响应式的对象

使用：1 从vue包中导入 reactive函数。 2. 在script setup中执行reactive函数并传入类型为对象的初始值，并使用变量接受返回值

```vue
<script setup>
	// 导入
	import {reactive} from 'vue'
    // 执行函数 传入参数 变量接受
    const state = reactive('对象类型数据')
</script>
```

案例

```vue
<script setup>
import {reactive} from 'vue'
    const state = reactive({
        count:0
    })
    const setCount = ()=>{
        state.count++
    }
</script>

<template>
	<div>
    	<button @click="setCount">
        	{{state.count}}    
    	</button>
    </div>
</template>
```

ref()

作用：接受简单类型或者对象类型数据 传入并返回一个响应式的对象

使用：1.从vue包中导入ref函数。 2.在script setup中执行ref函数并传入初始值，使用变量接受ref函数的返回值

```vue
<script setup>
	// 导入
	import {ref} from 'vue'
    // 执行函数 传入参数 变量接受
    const state = ref('简单类型数据或者复杂类型数据')
</script>
```

案例

```vue
<script setup>
    import {ref} from 'vue'
    const count = ref(0)
    
    // 使用函数修改值时需要使用.value属性进行修改
    const changeCount = ()=>{
        count.value++
    }
    
    // 如果传入的是一个对象
    const count2 = ref({
        m1:0,
        m2:1
    })
    
    const changeCount2 = ()=>{
        count.value.m1++
    }
</script>

<template>
	<div>
        <button @click>
            {{ count }}
    	</button>
    </div>
</template>
```

### 3.computed计算属性函数

计算属性基本思想和vue2基本一致，组合式API下的计算属性只是修改了写法

使用：1.导入computed函数。2.执行函数 在回调函数中return基于响应式数据做计算的值用变量接收

```vue
<script setup>
	//导入
    import {computed} from 'vue'
    //执行函数 变量接收 在回调函数中return 计算值
    const computedState = computed(
    ()=>{
        return '基于响应式数据做计算之后的值'
    }
    )
</script>
```

案例

```vue
<script>
	import {computed} from 'vue'
    // 声明变量
    const oldList = [1, 2, 3, 4, 5, 6, 7, 8]
    // 执行函数 变量接收 在回调函数中return 计算值
    const newList = computed(
        ()=>{
            return oldList.value.filter(
            	item => item > 2
            )
        }
    )
    // 得到的结果为：[3, 4, 5, 6, 7, 8]
    
    setTimeout(()=>{
        oldList.value.push([9,10])
    },3000)
    
    // 得到的寄过为：[3, 4, 5, 6, 7, 8, 9, 10]
</script>
```

- 注意：
  + 计算属性中不应该有副作用（比如异步请求/修改dom)
  + 避免直接修改计算属性中的值(计算属性应该是只读的)

### 3.watch函数

作用：侦听一个或多个数据的变化，数据变化时执行回调函数

两个额外参数：1.immediate(立即执行) 2.deep(深度侦听)

使用：

侦听单个数据

1.导入watch函数。2.执行watch函数传入要侦听的响应式数据(ref 对象)和回调函数

```vue
<script setup>
	// 导入函数
    import {ref, watch} from 'vue''
    const count = ref(0)
    // 调用watach函数 侦听变化
    watch(count, (newValue, oldValue) => {
        // ref对象不需要加.value
        console.log(`count发生了变化，旧值为${oldValue}, 新值为${newValue}`)  
    })
</script>
```

侦听多个数据

同时侦听多个响应式数据的变化，不管哪个数据变化都需要执行回调

```vue
<script setup>
	import {ref, watch} from 'vue'
    const count = ref(0)
    const name = ref('cp')
    
    // 侦听多个数据
    watch(
    [count, name], ([newCount, newName], [oldCount, oldName]) =>{
        console.log('count或者name变化了', [newCount, newName], [oldCount, oldName])
    }
    )
</script>
```

案例

```vue
<script setup>
import { watch, ref } from 'vue';

const count = ref(0)
const name = ref('cp')

const changeCount = () => {
  count.value++
}

const changeName = () => {
  name.value = name.value.split('').reverse().join('')
}

watch(
  [count, name], ([newCount, newName], [oldCount, oldName]) => {
    console.log('count 或者name的值发生了变化', [newCount, newName], [oldCount, oldName])
  }
)

</script>
```

immediate

说明：在侦听器创建时立即触发回调，响应式数据变化后继续执行回调

```vue
<script setup>
const count = ref(0)
watch(count, (newCount, oldCount)=>{
    console.log('count发生了变化')， {
        immediate: true
    }
})
</script>
```

deep

说明:在侦听对象，数组类数据类型时, 对其内部深度变化进行监听

```vue
<script setup>
    const state = ref({
        m1:1,
        m2:2
    })
    
    watch(state, ())
</script>
```

* 注意：deep性能损耗较大，尽量不开启deep

案例

```vue
<script setup>
    import {ref, watch} from 'vue'
    const state = ref({count:0})
    watch(state, (newValue, oldValue)=>{
        console.log('数据变化了，新的数据为：' + newValue)
        // 如果没有deep参数的话，直接修改对象中的数据是不会被侦听到的
    }, {deep: true})
    
    // 定义一个方法用于直接修改对象中的数据
    const changeState = () =>{
        state.value.count++
    } 
</script>
```

精确侦听

需求：在不开启deep的前提下，侦听age的变化只有age变化时才能执行回调

```vue
<script setup>
    import {ref, watch} from 'vue'
    const state = ref({count:0, age:2})
    

    watch(
        ()=>state.value.age, 
        // 将state中的属性用回调函数传入watch
        (newVlaue, oldValue)=>{
        console.log('数据变化了')
    })
    
    const changeAge = ()=>{
        state.value.age++
    }
</script>
```

* 疑问：如果想要监听同一对象中的两个或两个以上的属性时应该如何处理？

### 4.生命周期函数

|      选项式API       |    组合式API    |
| :------------------: | :-------------: |
| beforeCreate/created |      setup      |
|     beforeMount      |  onBeforeMount  |
|       mounted        |    onMounted    |
|     beforeUpdate     | onBeforeUpdate  |
|       updated        |    onUpdated    |
|    beforeUnmount     | onBeforeUnmount |
|      unmounted       |   onUnmounted   |

使用：1.导入生命周期函数。2.执行生命周期函数 传入回调

```vue
<script setup>
    import {onMounted} from 'vue'
    // 执行函数，并传递回调
    onMounted(()=>{
        console.log('组件已挂载')
    })
</script>
```

执行多次

生命周期函数是可以执行多次的，多次执行时传入的回调会在时机成熟时依次执行。

```vue
<script setup>
	import {onMounted} from 'vue'

    onMounted(()=>{
        console.log('1')
    })
    
    onMounted(()=>{
        console.log('2')
    })
    
    // 结果为
    // 1
    // 2
</script>
```

* <font color='green'>可以用于补充代码</font>

### 5.父子通信 - (父传子)

基本思想

1.父组件给子组件绑定属性

2.子组件内部通过props选项接受

案例

* 父组件

```vue
<script setup>
// 导入子组件
import sonComVue from './son-com.vue'
const count = ref(100)
setInterval(()=>{
    count.value++
},3000)
</script>

<template>
<!--绑定属性以及响应式数据-->
<sonComVue message="this is app message" :count="count"/>
</template>
```

* 子组件

```vue
<script setup>
// 通过defineProps "编译器宏" 接受数据
const props = defineProps({ message:String, count:Number })
console.log(props)
</script>

<template>
{{message}} - {{count}}
</template>

```

### 6.父子通信 - (子传父)

基本思想

1.父组件中给子组件标签通过@绑定事件

2.子组件内部通过$emit方法触发事件

案例

父组件

```vue
<script setup>
// 导入子组件
import sonComVue from './son-com.vue'
const getMessage = (msg)=>{
    console.log(msg)
}
</script>

<template>
<!--绑定自定义事件-->
<sonComVue @get-message="getMessage" /></sonComVue>
</template>
```

子组件

```vue
<script setup>
    // 通过defineEmits编译器宏生成emit方法
    const emit = definEmits(['get-message'])
    const sendMsg = ()=>{
        // 触发自定义事件 并传递参数
        // 第二个参数为数据
        emit('get-message', 'this is son msg')
    }
</script>

<template>
	<button @click="sendMsg">sendMsg</button>
</template>
```

* 疑问：如果emit中有多个事件，应该如何处理

### 7.模板引用

概念

通过ref标识获取真实dom对象或者组件实例对象

使用

1.调用函数生成一个ref对象。2.通过ref标识绑定ref对象到标签

案例

获取dom实例对象

```vue
<script setup>
import {ref, onMounted} from 'vue'
// 调用ref函数得到ref对象
const h1Ref = ref(null)

onMounted(()=>{
    console.log(h1Ref.value)
})
</script>

<template>
<!--通过ref标识绑定ref对象-->
<h1 ref="h1Ref">I am a label of DOM named 'h1'</h1>
</template>
```

获取组件实例对象

```vue
<script setup>
import {ref} from 'vue'
import SonComVue from './son-com.vue'
const comRef = ref(null)

const showSon = ()=>{
    console.log(comRef.value)
}
</script>

<template>
	<button @click="showSon">
        click me
    </button>
	<SonComVue ref='comRef'></SonComVue>
</template>
```

defineExpose()

默认情况下在<script setup>语法糖下组件内部的属性和方法是不开放给父组件访问的，可以通过defineExpose编译宏指定哪些方法允许访问

案例

子组件

```vue
<script setup>
import {ref} from 'vue'
const name = ref('Mike')
const changeName = ()=>{
    name.value = name.value.split('').reverse().join('')
}

defineExpose({
    name, changeNmae
})
</script>
```

父组件

```vue
<script setup>
import {ref} from 'vue'
import SonComVue from './son-com.vue'
const comRef = ref(null)
const switchName = () => {
    comRef.value.changeName()
}
</script>

<template>
<button @click='switchName'>
	click me    
</button>
<SonCom ref="comRef" />
</template>
```

### 8.provide和inject

作用：顶层组件向任意的底层组件传递<font color='red'>数据</font>和<font color='yellow'>方法</font>，实现跨层组件通信

使用：1.顶层组件通过provide函数提供数据。2.底层组件通过inject函数获取数据

顶层组件

```js
provide('key', 顶层组件中的数据) // key 为标识
```

底层组件

```js
const message = inject('key') // 找到key
```

案例

顶层组件

```vue
<script setup>
// 导入proviede
import {provide, ref} from 'vue'
import RoomMsgItem from './room-msg-item.vue'

// 组件嵌套关系:
// RoomPage -> RoomMsgItem -> RoomMsgComment

// 提供静态数据
provide('data-key', 'this is a message')
    
// 提供响应式数据
const count = ref(0)
provide('count-key', count)
    
// 提供方法
const countPlus = ()=>{
    count.value++
}
provide('count-plus', countPlus)
</script>

<template>
	<div class='page'>
        <!--顶层组件-->
        <RoomMsgItem></RoomMsgItem>
    </div>
</template>
```

底层组件

```vue
<script setup>
// 导入inject
import {inject} from 'vue'
// 接收数据
const roomData = inject('data-key')
const roomCount = inject('count-key')
// 接收方法
const changeData = inject('count-plus')
</script>

<template>
	<div class="comment">
        <!--底层组件-->
       	<div>
        	来自顶层组件的数据为: {{ roomData }}
    	</div>
        <div>
        	来自顶层组件的响应式数据为：{{ roomCount }}
    	</div>
        <div>
        	<button @click='changeData'>click me</button>    
    	</div>
    </div>
</template>
```

* 注意：一棵组件树中存在多个顶层和底层对应关系





