附录A
=====

## 动态作用域

在第2章中，我们对比了动态作用域和词法做同于模型，JavaScript中的作用域就是词法做同于（事实上大部分语言都是基于词法作用域的）。

我们会简要地分析一下动态作用于，重申它与从发作用于的区别。但实际上动态作用域是JavaScript另一个重要机制this的表亲，本书第二部分“this和对象原型”中会有详细介绍。

从第2章中可知，此法作用于是一套关于引擎如何寻找变量以及会在何处找到变量的规则。此法作用于最重要的特征是它的定义过程在代码的书写阶段（假设你没有使用eval()或with）。

动态作用域似乎暗示有很好的理由让作用域作为一个在运行时就被动态确定的形式，而不是在写代码时进行静态确定的形式，事实上也是这样的。我们通过示例代码来说明：

```js

function foo() {
	console.log( a ); // 2
}

function bar() {
	var a = 3;
	foo();
}

var a = 2;
bar();

```

此法作用于让foo() 中的a通过RHS引用到了全局作用域中的a,因此会输出2。而动态作用域并不关心函数和作用域是如何声明以及在何处声明的，只关心他们从何处调用。换句话说，作用域链是基于调用栈的，而不是代码中的作用域嵌套。

因此，如果JavaScript具有动态作用域，理论上，下面的代码中的foo()在执行时将会输出3。

```js

function foo() {
	console.log( a ); // 3 (不是2！)
}

function bar() {
	var a = 3;
	foo();
}

var a = 2;
bar();

```

为什么会这样？因为当foo()无法找到a的变量引用时，会顺着调用栈在调用foo()的地方查找a，我不是在嵌套的此法作用于链中向上查找。由于foo()是在bar()中调用的，引擎会检查bar()的作用域，并在其中找到值为3的变量a。

很奇怪吧？现在你可能会这么想。

但这其实是因为你可能只读过基于此法作用于的代码（或者至少以词法作用域为基础进行了深入的思考），因此对动态作用域感到陌生。如果你只有基于动态作用域的语言写过代码，就会觉得这是很自然的，而词法自欧用于看上去才怪怪的。

需要明确的是，事实上JavaScript并不具有动态作用域。它只有此法作用域，简单明了。但是this机制某种程度上很像动态作用域。

主要区别：词法作用域是写在代码或者说定义时确定的，而动态作用域是在运行时确定的。（this也是！）词法作用域关注函数在何处声明，而动态作用域关注函数从何处调用。

最后，this关注函数如何调用，这就表明了this机制和动态作用域之间的关系多么紧密。如果想了解更多关于this的详细内容，参见本书的第二部分“this和对象原型”。

## 附录B 块作用域的替代方案

第3章深入研究了块作用域。失少从ES3发布以来，JavaScript中就有了块作用域，而with和catch分句就是块作用域的两个小例子。

但随着ES6中引入let，我们的代码终于有了创建完整，不受约束的块作用域的能力。块作用域在功能上和代码风格上都拥有很多激动人心的新特性。

但如果我们想在ES6之前的环境中使用块作用域呢？

考虑下面的代码：

```js

{
	let a = 2;
	console.log( a ); // 2
}

console.log( a ); // ReferenceError

```

这段代码在ES6环境中可以正常工作。但是在ES6之前的环境中如何实现这个效果？答案就是catch。

```js

try{throw 2;}catch(a){
	console.log( a ); // 2
}

console.log( a ); // ReferenceError

```

天啊这些代码既丑陋又奇怪。我们看见一个会强制抛出错误的try/catch，但是它会抛出的错误就是一个值2，然后catch分句中的变量声明会接收到这个值。头疼！

没错，catch分句具有块作用域，因此它可以在ES6之前的环境中作为块作用域的替代方案。

“但是”，你可能会说，“鬼才要写这么丑陋的代码！”没错，没人写的代码像CoffeeScript编译器输出的代码，但这不是重点。

重点是工具可以将ES6的代码转换成能在ES6之前环境中运行的形式。你可以使用块作用域来写代码，并享受它代码来的好处，然后在构建时通过工具来对代码进行预处理，使之可以在部署时正常工作。

事实上，这是向ES6中的所有（好吧，不是所有而是大部分）功能迁移的首选方式：在从ES6之前的环境向ES6过渡是，使用代码转换工具来对ES6代码进行处理，生成兼容ES5的代码。

## B.1 Traceur

Google维护着一个名为Traceur的项目，该项目正是用来将ES6代码转换为兼容ES6之前的环境（大部分是ES5，但不是全部）。TC39委员会依赖这个工具（也有其他工具）来测试他们指定的语义化相关的功能。

Traceur会将我们的代码片段转换成什么样子？你能猜到的！

```js

{
	try {
		throw undefined;
	} catch (a) {
		a = 2;
		console.log( a );
	}
}

console.log( a );

```

通过使用这样的工具，我们就可以在使用块作用域时无需考虑目标平台是否是ES6环境，因为try/catch从ES3开始就存在了（并且一直都是这样的）。

## B.2 隐式和显式作用域

在第3章中介绍块作用域时，我们的代码有一些可维护性和可扩展性方面的缺陷，有没有其他可以使用块作用域，并且还能避免这种缺陷的途径？

考虑下面这种let的使用方法，它被称作let作用域或let声明（对比前面的let定义）。

```js

let (a = 2) {
	console.log( a ); // 2
}

console.log( a ); // ReferenceError

```

同隐式地劫持一个已经存在的作用域不同，let声明会创建一个现实的作用域并与其进行绑定。显示作用域不仅更加突出，在代码重构时也表现得更加健壮。在语法上，通过强制性地将所有变量声明提升到块的顶部来产生更简洁的代码。这样更容易判断变量是否属于某个作用域。

这种模式同很多人在函数作用域中手动将var声明提升到函数顶部的方式很接近。let声明有意将变量声明放在块的顶部，如果你并没有到处使用let定义，那么你的块作用域就很容易辨识和维护。




