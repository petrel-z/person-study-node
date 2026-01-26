原理：
每个对象内部都有一个`[[class]]`属性，用来记录该对象的真实类型。
Object.prototype.toString()是获取对象内部的`[[class]]`属性
返回调用toString()方法的对象的`[[class]]`属性，this指向Object.prototype，所以返回`[object object]`
call()是js函数中的方法，它的作用是调用一个函数，并指定该函数运行时内部的this指向
Object.prototype.toString.call()中call()方法将Object.prototype.toString函数运行时的this指向传入的参数，故在执行中，就会读取传入参数的`[[class]]`属性

```js
const arr = [1, 2, 3];
const str = "hello";
const num = 42;
const reg = /abc/;
const date = new Date();

console.log(Object.prototype.toString.call(arr));  // 输出 [object Array]
console.log(Object.prototype.toString.call(str));  // 输出 [object String]
console.log(Object.prototype.toString.call(num));  // 输出 [object Number]
console.log(Object.prototype.toString.call(reg));  // 输出 [object RegExp]
console.log(Object.prototype.toString.call(date)); // 输出 [object Date]
```
[[null的历史遗留问题]]