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

### 2. reactive和ref函数

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

