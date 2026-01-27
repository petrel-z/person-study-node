this是什么？
this是函数的上下文对象

为什么要有this？
当函数调用时，操作对象时更加方便

this绑定有哪些方法？按照优先级顺序
1. 构造函数的new 实例在创建时，this指向这个实例
2. 显示绑定，通过bind/call/apply等方式改变函数的this指向
3. 隐式绑定，通过对象调用对象中的函数，this指向调用该函数的对象
4. 默认绑定，函数内部的this默认指向window，严格模式为undefined
