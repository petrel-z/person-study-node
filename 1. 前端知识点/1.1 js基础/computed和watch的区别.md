
## vue3响应式原理
把Vue响应式数据原理想象成一个餐厅的运营模式：

- **餐厅老板（`Object.defineProperty`）**：餐厅老板特别细心，会留意每桌客人点了什么菜。只要客人点菜或者退菜，老板马上就能知道并做出安排。在Vue里，`Object.defineProperty` 就像这个老板，能拦截对象属性的读取和赋值操作。
- **菜单（`data` 对象）**：菜单上列着各种菜品，每一道菜就相当于 `data` 对象里的一个属性。
- **客人（组件）**：来餐厅吃饭的客人会看菜单点菜，他们就好比Vue的组件，会读取 `data` 对象里的属性值。
- **订单记录（`Dep` 对象）**：老板有一个本子，专门记录每道菜都被哪些客人点了。这个本子就相当于 `Dep` 对象，负责收集依赖，也就是记录哪些组件依赖了某个属性。
- **上菜通知（`Watcher` 对象）**：当一道菜做好之后，老板会根据本子上的记录，通知点了这道菜的客人。客人收到通知后，就可以享用美食了。在Vue里，`Watcher` 对象就像上菜通知，当属性值发生变化时，会通知所有依赖这个属性的组件进行更新。

依赖收集——》 触发更新

```js
// 模拟Dep对象
class Dep {
    constructor() {
        this.subscribers = [];
    }
    // 收集依赖
    depend() {
        if (Dep.target) {
            this.subscribers.push(Dep.target);
        }
    }
    // 通知更新
    notify() {
        this.subscribers.forEach(watcher => watcher.update());
    }
}

Dep.target = null;

// 模拟Watcher对象
class Watcher {
    constructor(getter) {
        this.getter = getter;
        this.get();
    }
    get() {
        Dep.target = this;
        this.getter();
        Dep.target = null;
    }
    update() {
        console.log('数据更新啦');
    }
}

// 模拟Object.defineProperty
function defineReactive(obj, key, val) {
    const dep = new Dep();
    Object.defineProperty(obj, key, {
        get() {
            dep.depend();
            return val;
        },
        set(newVal) {
            if (newVal!== val) {
                val = newVal;
                dep.notify();
            }
        }
    });
}

// 创建对象
const data = {
    fruit: '苹果'
};

// 让对象属性变成响应式
defineReactive(data, 'fruit', data.fruit);

// 创建Watcher
new Watcher(() => data.fruit);

// 修改属性值
data.fruit = '香蕉';

```

### 具体流程

1. **餐厅开业（Vue实例创建）**：老板对菜单上的每道菜进行登记（用 `Object.defineProperty` 对 `data` 对象的属性进行拦截），准备好记录订单的本子（创建 `Dep` 对象）。
2. **客人点菜（组件读取属性值）**：老板在本子上记下客人点了哪道菜（`Dep` 对象收集依赖）。
3. **菜品更新（属性值发生变化）**：老板根据本子上的记录，通知点了这道菜的客人（`Dep` 对象通知所有 `Watcher` 对象），客人收到通知后开始用餐（组件更新）。

## 二者的定义：
computed: 
基于响应式数据派生出新的响应式数据，具有缓存特性，只有当依赖的源数据变化时才会重新计算。
watch:
监听一个或多个响应式数据的变化，当数据变化时执行自定义逻辑（如异步操作、数据持久化等）。

## 使用方法：
computed: 

```javascript
import { ref, computed } from 'vue'

const count = ref(1)

// 简写
const doubleCount = computed(() => {
  return count.value * 2
})

// 对象式
const fullName = computed({
  get: () => `${firstName.value} ${lastName.value}`,
  set: (newValue) => {
    const [f, l] = newValue.split(' ')
    firstName.value = f
    lastName.value = l
  }
})
```

`computed` 会缓存计算结果，**只有当依赖的响应式数据变化时才会重新计算**，而普通方法每次调用都会执行。
`computed` 会自动追踪内部使用的响应式数据（`ref`、`reactive` 等），仅当这些依赖变化时才更新。

**它追踪其函数体内所有被访问过的响应式数据**
```javascript
const a = ref(1) 
const b = ref(2) 

const result = computed(() => {
  console.log('访问了 a 的值：', a.value)
  return b.value
})

console.log(result.value) // 输出：2（首次计算，访问 a 和 b）

a.value = 100
console.log(result.value) // 输出：2（重新执行，打印 "访问了 a 的值：100"）

b.value = 200
console.log(result.value) // 输出：200（重新执行，打印 "访问了 a 的值：100"
```


watch:
用于监听响应式数据的变化并执行副作用（如数据处理、异步操作等）
```javascript
import { ref, watch } from 'vue'

const count = ref(0)
const user = ref({ name: 'vue' })

// 监听单个 ref
watch(count, (newVal, oldVal) => {
  console.log(`count 从 ${oldVal} 变为 ${newVal}`)
})

// 监听对象的某个属性（需用函数返回）
watch(
  () => user.value.name,
  (newVal) => {
    console.log(`name 变为 ${newVal}`)
  }
)

// 监听多个源
watch([count, () => user.value.name], ([newCount, newName], [oldCount, oldName]) => {
  console.log('多个源变化了')
})

// 立即执行（初始化时就触发一次）
watch(count, (newVal) => {}, { immediate: true })

// 深度监听（监听对象内部属性变化）
watch(user, (newVal) => {}, { deep: true })
```
注意：默认情况下，`watch` 不会监听对象内部嵌套属性的变化，需开启 `deep: true`。


## 二者的原理：
### computed：
1. **定义计算逻辑**：通过一个函数指定 “如何从已有响应式数据计算出新值”（比如从 `a` 和 `b` 计算出 `sum = a + b`）。
    
2. **依赖收集与计算**：初始化时，`computed` 会执行一次计算函数，在执行过程中读取到的响应式数据（如 `a` 和 `b`）会被 “收集为依赖”—— 即告诉这些数据：“我依赖你们，你们变了要通知我重新计算”。同时，第一次计算的结果会被缓存起来。
    
3. **缓存与更新**：
    - 当依赖的响应式数据**没有变化**时，多次访问 `computed` 属性会直接返回**缓存的旧结果**，不会重新执行计算函数（提升性能）。
    - 当依赖的响应式数据**发生变化**时，会触发 `computed` 的更新机制，重新执行计算函数得到新结果，并更新缓存，同时通知依赖这个计算属性的组件或其他响应式数据进行更新。

### watch：
Vue3 的响应式系统通过 `Proxy`（代理）劫持响应式数据（如 `ref`、`reactive` 创建的数据）的读写操作。`watch` 的工作流程可以拆解为 3 步：

1. **指定 “监控目标”**：`watch` 需要明确告诉它要监听哪个 / 哪些响应式数据（如 `ref` 变量、`reactive` 对象的属性等）。
    
2. **收集依赖**：初始化时，`watch` 会执行一次 “依赖收集”—— 读取被监听的响应式数据，触发数据的 `get` 拦截器，将 `watch` 的回调函数注册为 “依赖”（即告诉数据：“我在监控你，变了记得通知我”）。
    
3. **触发更新**：当被监听的数据发生变化时，会触发数据的 `set` 拦截器，此时数据会遍历所有注册的依赖（包括 `watch` 的回调），执行回调函数，并传入新值和旧值。


# 区别

### 1. **设计目的不同**

- **`computed`：计算属性**  
    本质是**基于依赖的响应式数据派生新值**，专注于 “计算”。它会根据依赖自动推导结果，更像是一个 “动态的属性”，用于描述数据之间的映射关系。  
- **`watch`：监听器**  
    本质是**监听数据变化并执行副作用**，专注于 “响应变化”。它用于在数据变化时触发额外操作（如异步请求、DOM 操作、本地存储、控制台打印等），不直接产生新数据。  
    

### 2. **使用场景不同**

- **`computed` 适合**：
    - 从现有响应式数据中**派生新数据**（如格式化日期、拼接字符串、过滤列表）。
    - 需要**缓存计算结果**（依赖不变时，多次访问不会重复计算）。
    - 逻辑简单的同步计算（无异步操作）。

数据格式化 
数据过滤 / 筛选
缓存频繁使用的计算结果
    
    示例：
    
    ```javascript
    const total = computed(() => price.value * count.value) // 从价格和数量计算总价
    ```
    
      
    
- **`watch` 适合**：
    - 数据变化时执行**异步操作**（如接口请求、定时器）。
    - 数据变化时需要**复杂逻辑处理**（如多步骤数据转换、触发其他非响应式操作）。
    - 监听数据变化并执行**副作用**（如操作 DOM、日志记录）。

数据变化时发送网络请求
    示例：
    
    ```javascript
    watch(userId, async (newId) => {
      // 异步请求用户详情
      const res = await fetch(`/api/user/${newId}`)
      userInfo.value = res.data
    })
    ```
    
      
### 3. **执行机制不同**

- **`computed`：缓存 + 懒执行**
    - **缓存性**：只有当依赖的响应式数据变化时，才会重新计算；依赖不变时，直接返回缓存结果。
    - **懒执行**：计算属性在**首次访问时才会执行计算**，若从未被使用，即使依赖变化也不会执行。

- **`watch`：即时响应 + 无缓存**
    - **即时性**：监听的数据变化时，会**立即执行回调**（除非使用 `immediate: false`，默认是 `false`）。
    - **无缓存**：每次数据变化都会触发回调，无论结果是否被使用。

### 4. **语法与返回值不同**

- **`computed`**：
    - 函数式写法（只读）：`computed(() => { ... })`，返回一个**计算属性引用**（需通过 `.value` 访问）。
    - 对象式写法（读写）：`computed({ get() {}, set(newVal) {} })`，可通过 `set` 反向修改依赖数据。

- **`watch`**：
    - 语法：`watch(监听目标, 回调函数, 配置项)`，返回一个**停止监听的函数**。
    - 无返回值，回调函数的作用是处理副作用（不直接产生新数据）。
