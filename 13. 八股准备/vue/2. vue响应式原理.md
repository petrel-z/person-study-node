在没有数据响应式之前，是怎么进行视图更新的？
当数据变化时，是通过手动操作dom来实现视图的更新
缺点：
1. 代码不易维护，有大量的修改dom的代码
2. 性能差，频繁修改dom导致重排或重绘，消耗性能
3. 易出错，修改一处dom，其他相关该dom的地方都要改变

响应式实现：
1. 数据劫持
2. 依赖收集
3. 触发更新

有一处数据，收集使用这个数据的地方，当监听到数据改变时，通知引用该数据的地方

# vue2的响应式是如何实现的？
Object.defineProperty()

observe 观察者，用于数据劫持
dep，用于依赖手机，通知watch
watch 监听者，用于接受更新通知，并执行更新操作

缺点：
1. 无法监听到新增或删除的属性，必须手动触发vue.set 或者vue.delete
2. 无法深度监听对象属性，必须通过递归实现深度监听，数据复杂时，性能较差
3. 数组性能差

# vue3的响应式是如何实现的？
Proxy代理整个对象 + Reflect
解决了vue2中的响应式问题
可监听对象属性的新增和删除、数组性能差

无法代理基本数据类型，只能通过ref将基本类型包装成对象，从而实现响应式



# 代码

## vue2
```js
// 1. Dep：收集依赖、通知更新
class Dep {
  constructor() {
    this.subs = []; // 存储依赖的 Watcher
  }
  // 添加 Watcher
  addSub(watcher) {
    this.subs.push(watcher);
  }
  // 通知所有 Watcher 更新
  notify() {
    this.subs.forEach(watcher => watcher.update());
  }
}

// 2. Watcher：接收更新通知，执行视图更新
class Watcher {
  constructor(cb) {
    this.cb = cb; // 回调函数（比如更新视图的逻辑）
    Dep.target = this; // 标记当前 Watcher，方便 Dep 收集
  }
  // 执行更新
  update() {
    this.cb();
  }
}

// 3. Observer：劫持数据，添加 getter/setter
class Observer {
  constructor(data) {
    this.walk(data); // 遍历数据
  }
  // 遍历对象属性（仅处理对象，数组单独处理）
  walk(obj) {
    Object.keys(obj).forEach(key => {
      this.defineReactive(obj, key, obj[key]);
    });
  }
  // 核心：给属性添加 getter/setter
  defineReactive(obj, key, val) {
    const dep = new Dep(); // 每个属性对应一个 Dep
    // 递归劫持子属性（深度监听）
    if (typeof val === 'object' && val !== null) {
      new Observer(val);
    }
    // 劫持属性
    Object.defineProperty(obj, key, {
      enumerable: true,
      configurable: true,
      // 读取属性时：收集依赖
      get() {
        // Dep.target 存在时（即有 Watcher 在读取该属性），收集依赖
        if (Dep.target) {
          dep.addSub(Dep.target);
        }
        return val;
      },
      // 修改属性时：触发更新
      set(newVal) {
        if (newVal === val) return;
        val = newVal;
        // 新值如果是对象，递归劫持
        if (typeof newVal === 'object' && newVal !== null) {
          new Observer(newVal);
        }
        dep.notify(); // 通知所有 Watcher 更新
      }
    });
  }
}

// 测试：模拟 Vue 响应式
const data = { name: '张三' };
// 1. 劫持数据
new Observer(data);
// 2. 创建 Watcher（模拟视图依赖）
new Watcher(() => {
  console.log('视图更新：', data.name);
});
// 3. 读取数据 → 触发 get，收集依赖
console.log(data.name); // 输出「视图更新：张三」+「读取了 name 属性」
// 4. 修改数据 → 触发 set，通知 Watcher 更新
data.name = '李四'; // 输出「视图更新：李四」
```

## vue3
```js
const obj = { name: '张三', age: 18 };
// 创建 Proxy 代理
const proxyObj = new Proxy(obj, {
  // 读取属性（包括 obj.name、obj['age']、数组下标）时触发
  get(target, key, receiver) {
    console.log('读取：', key);
    // Reflect 执行原生读取操作，保证行为一致
    return Reflect.get(target, key, receiver);
  },
  // 赋值（包括新增属性）时触发
  set(target, key, value, receiver) {
    console.log('修改/新增：', key, value);
    // Reflect.set 返回布尔值，标识操作是否成功
    return Reflect.set(target, key, value, receiver);
  },
  // 删除属性时触发
  deleteProperty(target, key) {
    console.log('删除：', key);
    return Reflect.deleteProperty(target, key);
  }
});

proxyObj.name; // 触发 get → 输出「读取：name」
proxyObj.age = 20; // 触发 set → 输出「修改/新增：age 20」
proxyObj.gender = '男'; // 触发 set → 输出「修改/新增：gender 男」（支持新增属性）
delete proxyObj.age; // 触发 deleteProperty → 输出「删除：age」（支持删除属性）
```