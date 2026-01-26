### ***防抖函数***
``` js
const debounceInput = document.querySelector('.debounce');
// 防抖函数
function debounce(fn, delay) {
  // 设置定时器
  let timer;
  return function (...args) {
    clearTimeout(timer);
    timer = setTimeout(() => {
      // fn.apply(this, args);
      fn(...args);
    }, delay)
  }
}
const handleInput = debounce(search, 1000);
debounceInput.addEventListener('keyup', function (e) {
  handleInput(e.target.value);
});
```

### ***节流函数***
```js
function throttle(fn, delay) {
    let lastTime = 0;  // 上次执行时间
    
    return function(...args) {
        const now = Date.now();
        
        // 若距离上次执行超过 delay，则执行函数
        if (now - lastTime >= delay) {
            lastTime = now;
            fn.apply(this, args);
        }
    };
}
// 原始函数：打印 this 指向
function handleClick() {
  console.log('this 指向:', this);
}

// 创建节流函数
const throttledClick = throttle(handleClick, 1000);

// 给按钮绑定点击事件
const btn = document.querySelector('button');
btn.addEventListener('click', throttledClick);

```

***节流函数的原理***

***this指向的变化***
谁调用这个函数，这个函数内部的this指向就指向谁
在全局作用域调用函数fn() fn函数内部的this就指向window
在节流函数中



[[闭包]]
