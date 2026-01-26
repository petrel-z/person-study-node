以下是JavaScript中`this`指向六种规则的具体示例：

### 1. **全局作用域**：`this`指向全局对象（浏览器中是`window`）
```javascript
console.log(this === window); // true

// 全局变量会成为window的属性
var globalVar = 'Hello';
console.log(window.globalVar); // 'Hello'
console.log(this.globalVar);   // 'Hello'

// 函数内部的this（非严格模式）
function test() {
  console.log(this === window); // true
}
test();
```


### 2. **函数调用**：`this`指向全局对象（严格模式下是`undefined`）
```javascript
// 非严格模式
function regularFunction() {
  console.log(this === window); // true
}
regularFunction();

// 严格模式
function strictFunction() {
  'use strict';
  console.log(this === undefined); // true
}
strictFunction();

// 嵌套函数的this
function outer() {
  function inner() {
    console.log(this === window); // true（非严格模式）
  }
  inner();
}
outer();
```


### 3. **方法调用**：`this`指向调用该方法的对象
```javascript
const person = {
  name: 'Alice',
  greet: function() {
    console.log(`Hello, ${this.name}`); // 'Hello, Alice'
  },
  nested: {
    name: 'Bob',
    sayHi: function() {
      console.log(`Hi, ${this.name}`); // 'Hi, Bob'
    }
  }
};

person.greet();         // this指向person
person.nested.sayHi();  // this指向nested对象

// 注意：方法被赋值后，this可能改变
const greetFunc = person.greet;
greetFunc(); // 'Hello, undefined'（非严格模式下this指向window）
```


### 4. **构造函数**：`this`指向新创建的实例
```javascript
function Animal(name) {
  this.name = name;
  this.speak = function() {
    console.log(`${this.name} makes a sound.`);
  };
}

const dog = new Animal('Buddy');
dog.speak(); // 'Buddy makes a sound.'
console.log(dog instanceof Animal); // true
```


### 5. **箭头函数**：`this`继承自外层作用域
```javascript
const obj = {
  name: 'Alice',
  regular: function() {
    console.log(this.name); // 'Alice'
  },
  arrow: () => {
    console.log(this.name); // undefined（this继承自window）
  },
  nested: {
    name: 'Bob',
    getArrow: function() {
      return () => console.log(this.name); // 'Bob'（继承自getArrow的this）
    }
  }
};

obj.regular();      // 'Alice'
obj.arrow();        // undefined
obj.nested.getArrow()(); // 'Bob'

// 常见陷阱：箭头函数在全局作用域中
const timer = {
  seconds: 10,
  start: function() {
    setInterval(() => {
      this.seconds--; // this继承自start方法的this（即timer对象）
      console.log(this.seconds);
    }, 1000);
  }
};

timer.start(); // 每秒减少1，正确
```


### 6. **显式绑定**（`call`/`apply`/`bind`）：`this`指向第一个参数
```javascript
const person = {
  name: 'Alice'
};

function greet(message) {
  console.log(`${message}, ${this.name}`);
}

// call方法：参数逐个传递
greet.call(person, 'Hello'); // 'Hello, Alice'

// apply方法：参数作为数组传递
greet.apply(person, ['Hi']); // 'Hi, Alice'

// bind方法：创建新函数，永久绑定this
const greetAlice = greet.bind(person, 'Hey');
greetAlice(); // 'Hey, Alice'

// 实际应用：修复回调函数的this
const button = document.querySelector('button');
button.addEventListener('click', function() {
  this.disabled = true; // this指向button元素
}.bind(button));
```


### 总结
- **全局作用域**：`this`是`window`（浏览器）
- **函数调用**：`this`取决于是否启用严格模式
- **方法调用**：`this`是调用该方法的对象
- **构造函数**：`this`是新创建的实例
- **箭头函数**：`this`继承自外层作用域
- **显式绑定**：`this`由`call`/`apply`/`bind`的第一个参数决定

理解`this`的关键是**看函数如何被调用**，而不是在哪里定义。箭头函数是个例外，它的`this`由定义时的外层作用域决定。