# 一个关于 this 绑定的题目解析

## 题目内容

请写出输出内容

```
var fullname='aaa';

var obj = {
  fullname: 'bbb',
  getFullName: () => this.fullname,
  prop:{
    fullname: 'ccc',
    getFullName: function(){
      return this.fullname;
    }
  }
};

console.log(obj.prop.getFullName());

console.log(obj.getFullName());

var func1 = obj.prop.getFullName;
console.log(func1());

var func2 = obj.getFullName;
console.log(func2());
```

## 题目分析

很显然,这道题就是在考察**this 绑定**的问题。来复习一下关于 this 的知识点

`在绝大多数情况下，函数的调用方式决定了this的值。this不能在执行期间被赋值，并且在每次函数被调用时this的值也可能会不同。`

简单说就是**this 会指向调用者的对象**

so easy，答案很简单

第一，调用者是 obj.prop, 应该是 ccc

第二和第四，因为绑定了 obj，所以都是 bbb

第三，调用者是 window，所以是 aaa

```
 'ccc', 'bbb', 'aaa', 'bbb'
```

等等，是不是太顺利了，果然翻开答案一看

```
'ccc', 'aaa', 'aaa', 'aaa'
```

诶？箭头函数不是绑定父级作用域的么，难道出现了幻觉？为什么会绑定到 window 上呢？

先查一下 MDN，**_在箭头函数中，this 与封闭词法环境的 this 保持一致_**

那好，先来查一下封闭词法环境(enclosing lexical context)是什么东东。

`A function serves as a closure in JavaScript, and thus creates a scope, so that (for example) a variable defined exclusively within the function cannot be accessed from outside the function or within other functions`

`函数在JavaScript中被解析为一个闭包，从而创建了一个作用域，使其在一个函数内定义的变量不能从函数外访问或从其他函数内访问。`

那么难道是说**只有函数才能创建作用域**，其他变量都是不行的么？我们来验证一下

```
var fullname = "aaa";

var obj = {
  fullname: "bbb",
  prop: {
    fullname: "ccc",
    getFullName: () => this.fullname
  }
};
console.log(obj.prop.getFullName());

function A() {
  this.fullname = 'bbb';
}
var a = new A();
a.getFullName = () => this.fullname;
console.log(a.getFullName());

// 'aaa' 'aaa'
```

结果表明，不管是字面量还是 new 出来的**对象都是无法创建作用域的**

那么问题来了 this 的作用域在会随着上层函数的作用域变化而变化么？让我们来验证一下

```
var fullname='aaa';

var obj = {
  fullname: 'bbb',
  getFullName: function () {
      return (() => this.fullname)()
  }
};

console.log(obj.getFullName());
var getFullName = obj.getFullName;
console.log(getFullName());

// 'bbb', 'aaa'
```

答案是肯定的。好了，新知识 get 到手。

## 总结

箭头函数的 this 指向**父级函数的 this**，**对象无法创建作用域**

最后来复习一下关于 this 的相关知识，这里祭出**MDN 大法**

1. 在全局执行环境中（在任何函数体外部）this 都指向全局对象。
2. 在函数内部，this 的值取决于函数被调用的方式。
3. 当函数作为对象里的方法被调用时，它们的 this 是调用该函数的对象。
4. 当一个函数用作构造函数时（使用 new 关键字），它的 this 被绑定到正在构造的新对象。
5. 当函数被用作事件处理函数时，它的 this 指向触发事件的元素。
6. 当代码被内联 on-event 处理函数调用时，它的 this 指向监听器所在的 DOM 元素：
7. 在箭头函数中，this 与封闭词法环境的 this 保持一致
