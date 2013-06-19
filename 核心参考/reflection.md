# Reflection -- 反射 #

### 概述 ###

`Reflection` 是在运行时检查程序结构的一种流程，且可能动态修改程序结构。`Ruby` 中包含了一些很有用的反射特性，而 **JS.Class** 借鉴它们中的一些。

----------

### 对象属性(Object properties) ###

有些时候我们想要找出一个对象所属于的类，要么通过判断类型，要么通过调用类的方法来检测。所有通过JS.Class创建的对象都含有一个名叫 **klass** 的属性，它会指向对象所属的类：

	var Foo = new JS.Class();
	var obj = new Foo();
	
	obj.klass === Foo
	Foo.klass === JS.Class

所有的类都是 **JS.Class** 类的实例，这点与Ruby相同。另外，所有对象都带有一个 `isA()` 的方法。 当满足以下任意一个条件时， **obj.isA(Foo)** 返回 `true` ：

- `obj` 是 **Foo** 类的一个实例，或者是 **Foo** 的任意子类

- `obj` 是一个包含 **Foo** 模块的类的实例

- `obj` 是通过模块 **Foo** 进行扩展的

> 请记住，在Ruby中模块和类也都是对象，因此它们同样含有所有对象都有的标准方法。


----------

### 模块和类的反射(Module and class reflection) ###

模块和类都有一批方法可以支持检查继承树，检查方法搜查过程和提取特殊方法等。下面看一下创建的模块在反射上的使用：

	var ModA = new JS.Module({
	    speak: function() {
	        return "speak() in ModA";
	    }
	});
	
	var ModB = new JS.Module({
	    speak: function() {
	        return this.callSuper() + ", speak() in ModB";
	    }
	});
	
	var ModC = new JS.Module({
	    include: ModB,
	    speak: function() {
	        return this.callSuper() + ", and in ModC";
	    }
	});
	
	var Foo = new JS.Class({
	    include: [ModA, ModC],
	    speak: function() {
	        return this.callSuper() + ", and in class Foo";
	    }
	});

`ancestors()` 方法返回一个包含继承树上所有的类和模块的列表，列表中的首位是‘最远的’祖先。 **JS.Class**当做方法查找时是按照返回列表的逆序来进行搜索的。

	Foo.ancestors()
	// -> [JS.Kernel, ModA, ModB, ModC, Foo]

最后，可以使用 `instanceMethod()`方法从模块中提取出单一命名的方法，而使用 `instanceMethods`方法则会返回一组包含所有类实例方法的列表。当调用 `instanceMethods(false)`时，返回的方法只来自当前类或模块，忽略继承来的方法。如果要获取单一对象的所有定义的方法可以选择 `methods()` 方法。

	odC.instanceMethod('speak')
	// -> #<Method>
	
	Foo.instanceMethods()
	// -> ["speak", "__eigen__", "equals", "extend", "hash",
	//     "isA", "method", "methods", "tap", "wait", "_",
	//     "enumFor", "toEnum"]
	
	Foo.instanceMethods(false)
	// -> ["speak"]
	
	var f = new Foo();
	f.methods()
	// -> ["speak", "__eigen__", "equals", "extend", "hash",
	//     "isA", "method", "methods", "tap", "wait", "_",
	//     "enumFor", "toEnum"]

> 通过上面的代码可以发现，使用instanceMethods()方法与对象的methods()方法返回的结果一致。

----------

### 方法对象(Method objects) ###

**Module#instanceMethod()** 方法不会返回空的函数的；而是返回一个 `Method` 对象。该对象是一个类，JS.Class通过它来描述存储在模块中的方法，同时比空函数的优势在其可以为方法提供更多的上下文环境信息。

每个 **Method** 对象都含有以下特性：

- `module` - 定义了方法的模块或类

- `name` - 方法名称

- `arity` - 方法实参数量

- `callable` - 方法的回调函数

因此，比如当你从一个类中获得一个方法时，可以通过如下方式来检验该方法是否为另一个方法：

	klass.instanceMethod('foo').module

类似于JS中的函数，方法对象都可以响应 `call()` 和 `apply()` 方法，所以我们也可以考虑整合到方法中并支持回调函数的使用。

----------

### The eigenclass ###

所有的对象，包括模块和类都通过所谓的 `eigenclass` 来存储它们的 **单体方法** 。在Ruby中，这个eigenclass是个真正的类，但是在JS.Class中它作为一个模块来存在。(当你使用时未打算进行实例化或子类化则区别并不重要)任意的对象可以通过调用 `__eigen__()` 方法来访问其eigenclass。例如，你可以使用eigenclass来检查被继承方法的调用顺序：

	var obj = new Foo();
	obj.__eigen__().lookup('speak')
	// -> [#<Method>, #<Method>, #<Method>, #<Method>]

## END ##