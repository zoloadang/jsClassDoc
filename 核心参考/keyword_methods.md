# Keyword methods -- 关键词方法 #

### 概述 ###

使用JS.Class定义的方法都会在源代码中清晰地罗列出来。例如，一个`BlogPost`的类含有一个名为`publish()`的方法，那么你可以直接在类体中进行定义：

	BlogPost = new JS.Class({
	    publish: function() { /* ... */ }
	});

> 关键字方法有一些不同之处：它们是动态生成同时其行为动作取决于当前方法调用时的上下文环境的方法。JS.Class提供了一部分内建的关键词方法方便于在任意方法中直接使用。


----------

### callSuper() ###

这个方法常用于从继承堆栈链上的下一个模块中调用当前方法(即调用父级中的同名方法)，关于在JS.Class中是如何查找方法的详情可以查看[继承](./Inheritance.md)一文。

 **callSuper() **方法会自动传递所有参数到父级同名方法中，除非你指定要覆盖参数。

	Parent = new JS.Class({
	    say: function(something) {
	        JS.Console.puts(something);
	    }
	});
	
	// Outputs "hello" 
	new Parent().say('hello');
	
	Child = new JS.Class(Parent, {
	    say: function(something) {
	        // Outputs value of `something`
	        this.callSuper();
	
	        // Outputs uppercase version of `something`
	        this.callSuper(something.toUpperCase());
	    }
	});

----------

### blockGiven() ###

如果当前方法显式指定了回调函数且被调用时执行该回调，则返回`True`。

	Foo = new JS.Class({
	    say: function(a,b) {
	        return this.blockGiven();
	    }
	});
	
	var foo = new Foo();
	foo.say('some', 'words') // -> false
	foo.say('some', 'words', function() {}) // -> true


----------

### yieldWith() ###

如果当前方法带有一个回调函数作为参数被调用时(类似于上面的blockGiven()方法)，`yieldWith()` 使用给定的参数调用回调函数。如果未提供回函数参数，则`yieldWith()`方法默认什么也不做。

>  **yieldWith() **将使用回调函数之后的参数(若提供的话)作为`this`关键字的上下文环境。

	Foo = new JS.Class({
	    say: function(a,b) {
	        this.yieldWith(a + b);
	    }
	});
	
	var foo = new Foo(), object = {};
	
	foo.say('some', 'words', function(result) {
	    // result == 'somewords'
	    // this   == object
	}, object);


----------

### 自定义关键词方法 ###

可以通过JS.Class提供的应用接口来创建自定义的内建关键词。但请谨记，关键词是属于方法函数而且其上下文环境隐式地转换为当前调用的方法对象；如果你想将一个普通方法转换为常用行为动作操作时，则可以选择使用关键词。

下面举一个简单的例子来介绍，例如我们想要创建一个返回当前方法参数个数的关键词方法，可以按照如下方式实现：

	JS.Method.keyword('numArgs', function(method, env, receiver, args) {
	    return function() {
	        return args.length;
	    };
	});

接下来，刚刚上面定义的关键词就可以在方法中使用了：

	Foo = new JS.Class({
	    say: function(a,b) {
	        return this.numArgs();
	    }
	});
	
	var foo = new Foo();
	foo.say('some')             // -> 1
	foo.say('some', 'words')    // -> 2

显然上面这个例子比较简单。通常都是使用`JS.Method.keyword()`方法提供一个关键词名称和执行函数。这个执行函数需要接收如下几个参数然后返回另一个函数方法来实现关键词：

- method - 这个 **Method **对象代表的是当前正被调用的方法；详情可参考[reflection](./reflection.md)

- env - 当前正被调用的方法所属的类或模块对象；由于[继承](./Inheritance.md)的关系当前方法所依赖的模块可能并不是直接方法定义的模块,可能是其父级模块等等。

- receiver - 指定方法调用所在的上下文对象。

- args - 当前调用方法的参数对象。

使用这个语境和参数可以生成很多函数配合当前方法调用做出各种各样很多有用的东西。例如一个稍复杂的例子，JS.Class实现`callSuper`关键词方法：

	JS.Method.keyword('callSuper', function(method, env, receiver, args) {
	    var methods    = env.lookup(method.name),
	        stackIndex = methods.length - 1,
	        params     = Array.prototype.slice.call(args);
	
	    return function() {
	        var i = arguments.length;
	        while (i--) params[i] = arguments[i]; //覆盖默认参数
	
	        stackIndex -= 1; //父级方法索引
	        var returnValue = methods[stackIndex].apply(receiver, params);
	        stackIndex += 1;
	
	        return returnValue;
	    };
	});