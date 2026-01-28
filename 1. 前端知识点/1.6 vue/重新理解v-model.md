`v-model` 是 Vue 中用于实现**双向绑定**的语法糖，它的核心作用是：**在父组件和子组件之间同步数据**，既可以让父组件向子组件传递数据（父传子），也可以让子组件向父组件传递数据（子传父），从而简化了父子组件之间的通信代码。

# 背景：
我们正常来说的子传父和父传子，一般是将父组件中的数据，通过props的方式传递给子组件，子组件接受后，进行展示或者其他操作。当子组件里发生了一些变化，如用户进行了某些操作（表单输入、按钮提交等），此时子组件需要通知父组件，要么是数据改变了，要么是通知父组件处理用户操作。子组件会通过emit的方式通知父组件，还可能会将改变的数据传递给父组件（emit时携带参数）

那么我们通过props和emit的具体写法是什么呢？

# v-model本质写法

## 子组件：
```vue
<!-- MyInput.vue -->
<template>
  <div>
    <label>自定义输入框：</label>
    <!-- 
      1. :value="modelValue" 
         将父组件通过 props 传递过来的值绑定到 input 的 value 属性上。
         这是“父传子”的关键一步。
    -->
    <!-- 
      2. @input="handleInput"
         监听 input 事件。当用户在输入框中输入时，这个事件会被触发。
    -->
    <input 
      type="text" 
      :value="modelValue" 
      @input="handleInput" 
      placeholder="请输入内容..."
    />
    <p>子组件内部当前值：{{ modelValue }}</p>
  </div>
</template>

<script setup>
import { defineProps, defineEmits } from 'vue';

// 1. 定义 props，接收来自父组件的数据
// 'modelValue' 是一个约定的名称，当在父组件使用 v-model 时会默认传递这个 prop
const props = defineProps({
  modelValue: {
    type: String,
    required: true
  }
});

// 2. 定义 emits，声明要触发的事件
// 'update:modelValue' 也是一个约定的事件名，v-model 会默认监听这个事件
const emit = defineEmits(['update:modelValue']);

// 3. 定义事件处理函数
// 当 input 事件被触发时，这个函数会执行
const handleInput = (event) => {
  // event.target.value 是 input 框中最新的 value
  const newValue = event.target.value;
  
  // 4. 关键一步：通过 emit 触发事件，并将新值作为参数传递出去
  emit('update:modelValue', newValue);
};
</script>

```

该子组件的作用：
1. 通过defineProps定义modelValue 这个prop来接受父组件的数据，用于展示
2. 将modelValue绑定到input的value属性上
3. input监听事件
	1. 用户进行了输入操作，改变了输入框原本的父组件传递的内容
4. 触发事件
	1. 这触发了input输入框的change事件，该事件用于通知父组件，‘你需要修改这个数据，需要修改成xx（在这里是newValue）’，


## 父组件：
```vue
<!-- Parent.vue -->
<template>
  <div>
    <h2>父组件</h2>
    <p>父组件当前数据：{{ message }}</p>
    <!-- 
      使用我们自定义的 MyInput 组件
      
      1. :modelValue="message"
         将父组件的 data `message` 通过 props 传递给子组件。
         这完成了“数据从父到子”的单向流动。
      
      2. @update:modelValue="handleUpdate"
         监听子组件派发的 `update:modelValue` 事件。
         当子组件的值改变时，这个事件会被触发，父组件的 `handleUpdate` 方法会执行。
    -->
    <MyInput 
      :modelValue="message" 
      @update:modelValue="handleUpdate"
    />
  </div>
</template>

<script setup>
import { ref } from 'vue';
import MyInput from './MyInput.vue';

// 1. 父组件内部管理的数据源
const message = ref('初始值');

// 2. 定义事件处理函数，用于接收子组件传递过来的新值
const handleUpdate = (newValueFromChild) => {
  
  // 3. 参数是子组件传过来的，以此更新父组件的数据
  message.value = newValueFromChild;
};

</script>
```
该父组件的作用：
1. 管理数据message，并传递给子组件（单向数据传递）
2. 通过@update:modelValue这个事件监听，接收子组件传递的通知或传递的数据
3. 更新数据


# v-model语法糖的简化写法：

以上写法等价于：

```vue
<!-- Parent.vue 中使用 v-model 语法糖 -->
<template>
  <MyInput v-model="message" />
</template>

// 子组件中写法不变
```

v-model语法糖帮你完成的事情是：
- `:modelValue="message"`
- `@update:modelValue="newValue => message = newValue"`
还省略了handleUpdate的定义，妙哉妙哉🤔

# v-model常见应用场景：

1. 原生表单输入或自定义表单输入的双向数据绑定
2. 复选框（checkbox）
	1. 单个复选框：绑定boolean值
	2. 多个复选框：绑定数组
3. 单选框（radio）：绑定单个值，用于多选一
4. 下拉框（select）
5. 开关组件（switch）
6. 评分组件（rating）

# 多个v-model的绑定

```vue
<!-- 子组件：UserForm.vue -->
<template>
  <div>
    <input 
      type="text" 
      :value="name" 
      @input="emit('update:name', $event.target.value)" 
      placeholder="姓名"
    />
    <input 
      type="number" 
      :value="age" 
      @input="emit('update:age', $event.target.value)" 
      placeholder="年龄"
    />
  </div>
</template>

<script setup>
import { defineProps, defineEmits } from 'vue';

// 接收多个 prop
const props = defineProps({
  name: { type: String, default: '' },
  age: { type: Number, default: 0 }
});

// 声明多个事件
const emit = defineEmits(['update:name', 'update:age']);
</script>
```

```vue
<!-- 父组件：Parent.vue -->
<template>
  <div>
    <UserForm v-model:name="username" v-model:age="userAge" />
    <p>姓名：{{ username }}</p>
    <p>年龄：{{ userAge }}</p>
  </div>
</template>

<script setup>
import { ref } from 'vue';
import UserForm from './UserForm.vue';
const username = ref(''); // 同步姓名
const userAge = ref(0);   // 同步年龄
</script>
```


# 总结

`v-model` 的核心价值是**简化双向绑定逻辑**