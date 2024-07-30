# 一、选项式API/组合式API


1. 选项式API:VUE2，export default中的data,methods,computed,watch等都属于选项式API，又叫配置式API，像是一个个的配置


2. 组合式API:VUE3中的script标签中添加setup即可使用组合式API，其中可同时定义数据，函数，监听，计算属性

# 二、响应式数据定义

### 所谓响应式数据是指在html标签中使用模板语法绑定数据的时候，修改一个变量的值页面能够响应而随之更改

1. ref：可定义基本类型数据与对象类型数据，定义的对象可整体修改

    ```js
    let name = ref('张三');
    ```

2. reactive：可定义对象类型数据，定义的对象可修改对象属性，不可整体修改，object.assign，该方法会将原值覆盖以达到修改目的。

    ```js
    let car = reactive({
    brand:"奥迪",
    price:100
    })
    ```

3. toRef：toRef用于将对象中的某个属性转换为一个响应式的 ref 对象。它可以用于在不解构对象的情况下，将对象的属性独立地进行响应式处理。

    ```js
    let marketPrice = toRef(car.price)
    ```


4. toRefs：toRefs 是用于将整个对象的所有属性都转换为 ref 对象的一个方法。它可以一次性将对象的每个属性都转换为 ref，方便在解构时保持响应性。
    ```js
    let {brand,price} = toRefs(car)
    let newCar = toRefs(car)
    ```


# 三、计算属性

###    计算属性中有两个函数，处理计算值使用get()，修改计算值使用set(),注意修改时是先使用函数修改，再在set()中拿到修改后的值进行修改。

# 四、监听

1. ref定义的基本类型数据：简单监听即可

2. ref定义的对象类型数据：

    - 简单监听，监听的是对象的地址，对象的属性发生变化时，简单监听时监听不到的，只有在整个对象发生给变化时才能监听到。
        ```js
        watch(person1,(newVal,oldVal)=>{
            console.log('监听到变化',newVal,oldVal)
        })
        ```

    - 要想监听对象类型的属性变化需要手动开启深度监听。
        ```js
        watch(person1,(newVal,oldVal)=>{
            console.log('监听到变化',newVal,oldVal)
        },{deep:true})
        ```

    - 在对对象类型进行深度监听时，监听到某个属性的变化，新旧值相同，原因在于原值已被修改，所以新旧值相同，但监听整个对象的时候，新旧值不同，原因在于对象变化时	地址值变化，仍可找到原地址值的旧值。
    
3. reactive定义的对象类型数据：

    - 默认开启深度监听，即，不手动开启深度监听亦可监听对象的属性变化。底层隐式的创建了深度监听。

4. 监听ref或reactive定义的对象类型数据中的某个属性：

    - 若该属性是基本类型数据，则需要getter函数：()=>{对象.属性}
        ```js
        //其中'()=>{return person4_1.value.age}'即为getter函数
        watch(()=>{
            return person4_1.value.age
        },(newVal,oldVal)=>{
            console.log('监听到变化',newVal,oldVal)
        })
        ```

    - 若该属性是对象类型数据，可直接监听，但推荐getter函数形式监听。

    - 在直接监听时，若对象整体发生修改，则监听不到，原因在于该对象地址值已经改变。新的地址中该对象没有发生变化。

    -在使用getter监听时，可监听到整个对象的变化，但监听不到对象属性的变化

    - 要想整体、属性都监听，使用getter + deep:true
    
5. 同时监听以上情况中的多个数据：将多个数据放进数组里，可包含多个getter函数

6. 深度监听：监听中的一个属性：deep，开启则进行深度监听，简单监听只有在整个对象变化的时候才能监听到，深度监听在对象中的某个属性发生变化时就能监听到。深度监听相当于监听对象的地址值。
7. 立即监听：立即监听在指定值创建的时候即可监听，所以在赋值时，监听出的旧值为undefind
8. getter函数：能返回一个值的函数

9. watchEffect无须指定监听对象，在回调函数中涉及到的值都会自动监听
    ```js
    watchEffect(()=>{
    if (person5.age > 20){
        console.log('watchEffect监听成功')
    }
    })
    ```

# 五、标签的ref属性
1. ref属性使用在普通的html标签上，获取到的是DOM元素
    ```js
    <div ref="father_div">父组件</div>
    <button @click="showLog">输出</button>

    ...

    let father_div = ref()
    function showLog(){
    console.log(father_div.value)
    }

    // 输出内容：<div>父组件</div>
    ```
2. ref属性使用在组件上，获取到的是组件实例
    ```js
    // 父组件
    <div>
        <div>父组件</div>
        <Ref_Prop_SonComponent ref="son_comp"/>
        <button @click="showLog">输出</button>
    </div>

    ...

    function showLog(){
        console.log(son_comp.value.a)
    }

    ...

    // 子组件
    let a = ref(123)

    defineExpose({a})


    // 输出内容：123

    ```


# 六、泛型

1. 泛型定义：
    ```js
    // 定义泛型
    export interface PersonObj {
        name:string,
        id:string,
        age:number
    }

    export type PersonList = PersonObj[];

    ```


2. 泛型使用


    ```js
    // 使用泛型定义数据
    let personList:PersonList = ([{
        name:'mike',id:'asd',age:123
    }])
    // 使用泛型定义响应式数据
    let personList = reactive<PersonList>([{
        name:'mike',id:'asd',age:123
    }])
    ```

# 七、父子组件通信

1. 父组件获取子组件数据：
    ```js
    // 父组件
    <div>
        <div>父组件</div>
        <Ref_Prop_SonComponent ref="son_comp"/>
        <button @click="showLog">输出</button>
    </div>

    ...

    function showLog(){
        console.log(son_comp.value.a)
    }

    ...

    // 子组件
    let a = ref(123)

    defineExpose({a})


    // 输出内容：123

    ```

2. 子组件获取父组件数据：
- 情况一：只接收时可用于模板语法，但log时会报错：a_f未定义
```js
// 父组件
  <div>
    <div >父组件</div>
    <Ref_Prop_SonComponent ref="son_comp" a_f="123" />
    <button @click="showLog">输出</button>
  </div>
// 子组件
<div>子组件{{a_f}}</div>

```

- 情况二：接收并储存
```js
// 父组件
  <div>
    <div >父组件</div>
    <Ref_Prop_SonComponent ref="a_f" a_f="1312312" />
    <button @click="showLog">输出</button>
  </div>

// 子组件
<div>子组件{{a_f}}</div>

// 接收并保存
let x = defineProps(['a_f'])

function FatherDataLog(){
  console.log(x)
}
// 输出内容：Proxy(Object) {a_f: '1312312'}

```


- props接收 选项
    - 限制类型

        ```js
        // 限制类型{List:PersonList}中可填多种类型
        defineProps<{List:PersonList}>()
        ```
    - 限制必要性
        ```js
        defineProps<{List?:PersonList}>()
        ```
    - 指定默认值
        ```js
        withDefaults(defineProps<{List?:PersonList}>(),{
            name:'mike',id:'asd',age:123
        })
        ```

# 八、组件的生命周期

### 生命周期：又称生命周期函数、生命周期钩子，钩子函数
1. 创建：生成一个组件，创建一个实例
2. 挂载：塞到页面上，或者塞到整个项目中，与整体发生耦合，显示在页面上
3. 更新：组件内容发生改变
4. 卸载/销毁：组件从页面中移除

### vue2中共四个阶段，每个阶段各有前后两个状态，共八个钩子 
1. 创建(beforeCreat,created)
2. 挂载(beforeMount,mounted)
3. 更新(beforeUpdate，updated)
4. 销毁(beforeDestory,destroyed)

### vue3
1. 创建(setup)
2. 挂载(onBeforeMounted,onMounted)
3. 更新(onBeforeUpdate，onUpdated)
4. 卸载(onBeforeUnmount,onUnmounted)

### 细节
1. 编写示例：onMounted(()=>{})
2. 页面渲染时，父子组件各生命周期函数触发顺序<br>
    父组件创建完毕<br>
    父组件挂载之前<br>
    子组件创建完毕<br>
    子组件挂载之前<br>
    子组件挂载完毕<br>
    父组件挂载完毕<br>
3. 父组件销毁时：<br>
    父组件更新之前<br>
    子组件卸载之前<br>
    子组件卸载完毕<br>
    父组件更新完毕<br>
4. 子组件销毁时：<br>
    子组件更新之前<br>
    子组件更新完毕<br>

<font color="red">****  父子组件卸载的钩子函数调用与预期不符，请尽快解决</font>

# 九、Hooks
### 单独创建js/ts文件，包含一个功能的数据和方法(包含钩子，计算属性)

创建、声明：

export default function(){
    数据，函数定义
    return 数据、函数
}
使用：

.vue中可使用解构赋值获取内容

# 十、路由（Route）

SPA(single page web application)单页面应用常使用路由

1. 路由定义
```js
import {createRouter, createWebHistory} from "vue-router";
import Router1 from './route_1.vue'
import Router2 from './route_2.vue'
import Router3 from './route_3.vue'

const router = createRouter({
    history: createWebHistory(),
    routes:[
        {
            path:'/home',
            component:Router1
        },{
            path:'/news',
            component:Router2
        },{
            path:'/about',
            component:Router3
        }
    ]
})
export default router
```

3. 路由使用

- main.ts中
    ```js
    import router from '../src/components/router_comp/Router'
    import { createApp } from 'vue'
    import App from './App.vue'

    const app = createApp(App)
    app.use(router)
    app.mount('#app')
    ```
- 调用页面中
```js
// RouterLink标签：点击跳转至该标签绑定的路由
<RouterLink to="/about">关于</RouterLink>

// router-view路由组件将渲染的到该位置，
<router-view></router-view>
```
- to的几种写法

```js
<RouterLink to="/about">关于</RouterLink> 
<RouterLink :to="{path:'/about'}">关于</RouterLink>
<RouterLink :to="{name:'关于'}">关于</RouterLink>
```

4. 路由工作模式
- history 模式： 美观，但需要后端配合处理路径问题
- hash 模式：兼容性好，但SEOu欧化较差

5. 路由传参 
- query
- params

6. 路由嵌套

7. 路由规则的props配置 

8. replace属性
路由默认push模式，push模式可退回，前进
replace模式，无法退回，前进

9. 编程式导航
- 父组件
    ```js
    <template>
    <ul>
        <li v-for="item in lists" :key="item.id">
        <button @click="showDetail(item)">{{ item.profile }}</button>
        </li>
    </ul>
    <router-view></router-view>
    </template>

    ...

    interface listInterface {
        id:string,
        content:string,
        date:string,
        profile:string
    }
    function showDetail(item:listInterface){
        router.push({
            name:'newsDetail',
            query:{
            id:item.id,
            content:item.content,
            date:item.date,
            profile:item.profile
            }
        })
    }

    ```
- 子组件
    ```js
    <ul>
        <li>{{query.id}}</li>
        <li>{{query.content}}</li>
        <li>{{query.date}}</li>
        <li>{{query.profile}}</li>
    </ul>
    ...
        const route =useRoute()
        const { query } = toRefs(route)
    ```


10. 路由重定向
- 指定路径重新定位到另一个路径
```js
const router = createRouter({
    history: createWebHistory(),
    routes:[
        {
            path:'/',
            redirect:'/home'
        },{
            name:'主页',
            path:'/home',
            component:Router1
        }
    ]
})
export default router
```


# 十一、Pinia：集中式状态（数据）管理

1. ### pinia 环境搭建



2. ### pinia 使用
- 创建store
![alt text](image-3.png)
- 使用store
![alt text](image-4.png)
    - 存储数据
    ![alt text](image-1.png)

    - 获取数据
    ![alt text](image.png)
    - 修改数据 
    1. 直接修改 操作简单
    countStore.sum + = 1 
    2. 批量修改,可一次性修改一个或多个，多个数据操作简单
    count.$patch({
        sum:sum += 1
    })
    3. action 有点在于其中的方法可以复用
    countStore.increment(value)

- ### 对store的数据结构赋值会使其失去响应式
直接toRef会将store内的所有内用toRef，包括方法
用storeToRefs，只操作数据 
![alt text](image-5.png)


    
3. ### getters
getters里的方法不许调用即可执行，相当于计算属性
![alt text](image-6.png)
![alt text](image-7.png)

4. ### subscribe
相当于监听
![alt text](image-8.png)


5. ### store组合式写法
![ ](image-9.png)


# 十二、 组件通信
### 1. props 父子组件之间传参
不要使用props一层层传递
- 父传子
    ![alt text](image-10.png)
    ![alt text](image-11.png)
- 子传父
    可通过方法将数据传过去
    ![alt text](image-13.png)
    ![alt text](image-12.png)
### 2. 自定义事件 custorm
典型用于子传父，常见：@click
$event
对于原生事件,$event就是事件对象
对于自定义事件，$event就是触发事件时，所传递的数据
![alt text](image-14.png)

### 3. mitt
任意组件间传递数据 类似于事件总线
组件卸载时要解绑该组件绑定的事件
all获取所有事件
emit触发事件
off解绑事件 
on绑定事件
![alt text](image-15.png)
![alt text](image-16.png)
![alt text](image-17.png)

### 4. v-model 
数据的双向绑定 
父子双向传递
![alt text](image-18.png)


![alt text](image-19.png)
![alt text](image-20.png)

### 5.$attrs
祖孙传值 需要通过子组件传递 所以祖向孙传递的值可由子加工
孙传祖通过方法
![alt text](image-21.png)
![alt text](image-22.png)
![alt text](image-23.png)


### 6.$refs, $parent

- $refs可获取所有子组件实例对象，可用于对所有子组件做同样操作
    ![alt text](image-25.png)
    ![alt text](image-26.png)

    ![alt text](image-27.png)

- $parent获取父组件
    ![alt text](image-28.png)
    ![alt text](image-29.png)

### 7. provide, inject
祖孙传值，不涉及子组件
provide 向后代传值


![alt text](image-31.png)
![alt text](image-32.png)

inject (注入)将先祖组件的数据注入到当前组件
inject可传默认值，当接收不到指定数据时使用
![alt text](image-33.png)

一次性传输数据，方法
![alt text](image-34.png)
![alt text](image-35.png)

### 8. 插槽 
- 默认插槽 
![alt text](image-36.png)

- 具名插槽
![alt text](image-37.png)

语法糖：
![alt text](image-38.png)

- 作用域插槽 
![alt text](image-39.png)




![alt text](image-40.png)



# 十三、其他
### 1. shallowRef
- shallowRef & shallowReactive 浅层响应
定义的数据只有浅层（第一层）可被修改，在只关注整体修改时适用，防止对象数据过大，绕开深度响应，消耗性能。例：接口返回的只作展示的数据

- readonly & shallowReadonly
readonly 生成一个指定的响应式对象生成的深层只读副本
但数据来源可修改，所以数据源修改时生成的只读数据也会变

![alt text](image-41.png)

shallowReadonly 生成的数据只有浅层只读副本

- toRaw 用于一个响应式对象的原始对象，返回的对象不再是响应式 
- markRaw 对普通对象使用，使其永远不能被响应式
![alt text](image-42.png)

- customRef 创建一个自定义ref，并对其依赖项跟踪和更新触发进行逻辑控制
![alt text](image-43.png)

- teleport 将包含的的内容塞到任意指定位置且逻辑、数据留在当前文件

- susupense 异步组件  setup中如果有一部任务，一部过程未结束时会影响页面渲染
![alt text](image-45.png)