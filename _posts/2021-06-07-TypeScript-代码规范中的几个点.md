---
title: "TypeScript 代码规范中的几个点"
layout: post
category: blog
author: ingerchao
date: 2021-06-07
tag: 
- NodeJS
- ESLint
---



### 变量声明

- var: 全局变量，**作用域为该语句所在的函数内**；
- let: 局部变量，作用域为该语句所在的代码块内；
- const: 常量，作用域与let相同，不可以改变 const 量的值；

> 若后面对变量的值不做改变，最好用 const 替代 let。

### 函数

```typescript
// 具名函数
function func(){
  // code
}
 
// 匿名函数
let func=function(){
  // code
};

// 箭头函数全都是匿名函数
let func=()=>{
  // code
};
```

TypeScript 中，可以使用具名函数创建一个对象，就类似于面向对象编程中的构造函数。

```typescript
function Person(name,age){
   this.name=name;
   this.age=age;
}
let admin=new Person("Inger",18);
console.log(admin.name);
console.log(admin.age);
```

在普通函数中，this总是指向调用它的对象，如果用作构造函数，this指向创建的对象实例。而**箭头函数本身不创建this**，它在声明时可以捕获其所在上下文的this供自己使用。

```typescript
var webName="捕获成功";
let func=()=>{
  console.log(this.webName);
};
func();
```

### foreach

在规范中看到推荐的 foreach 写法是这样的：

```typescript
const arr:string[]=['a','b','c']

arr.forEach((i) => {
  console.log(i);
});
```

### 字符串

为什么规范中建议**用单引号定义字符串而不是双引号**？



**比起用 `+` 拼接字符串，更推荐用 \` \` 定义字符串模版，**为什么？

- `` 双撇号可以实现直接换行；
- 当我们用字符串模板去调用一个方法时，字符串模板表达式中的值就会自动赋值给被调用方法中的参数。

```typescript
function test(template, name, age) {
    console.log(template);  
    console.log(name);      // name就是${Name}
    console.log(age);       // age就是${getAge()}
}
  
const Name = 'Inger';
function getAge() {
    return 23;
}
  
// 通过字符串模板的方式，可以实现字符串的拆分
console.log(`Hello ${Name}, 
	I'm ${getAge()} years old.`);
test`Hello ${Name}, 
	I'm ${getAge()} years old.`;
```

### 代码格式

- 缩进： 2个空格;
- 每行末尾要加分号；
- 注释后面要有空格或 tab；
- 代码块首尾不要有空行；
- 代码块中不要有三行以上的空白行；
- 文件末尾必须有空白行；
- 大括号内的内容是单行时，`{` 后和`}`前必须有一个空格：`{ content }`;

