# The Kernel module -- 内核模块 #

### 概述 ###

在Ruby中的 `Kernel` 模块中定义了所有对象都公用的方法。同样地， `JS.Kernel` 也是定义的所有对象(包括类和模块对象)都共享的方法。每一个通过JS.Class创建的对象都会共享这些内核方法，但是它们也会根据不同的对象类型可以被覆写的。

----------

#### object.\_\_eigen\_\_() ####

返回对象的 `元模块(metamodule)` ，一个被用来存储任意[单体方法](./singleton_methods.md)的模块赋值给对象。通过调用类或模块的 `__eigen__()`方法可以返回一个包含它们自己方法的模块。

----------

#### object.enumFor(methodName, *args) ####

根据对象使用提供的 `methodName` 参数和可选参数返回一个枚举器。例如，通过 **Enumerable#forEachCons** 来生成枚举器：

	forEachCons: function(n, block, context) {
	    if (!block) return this.enumFor('forEachCons', n);
	    // forEachCons implementation details ...
	}

----------

#### object.equals(other) ####

当对象与提供的对象相等时返回 `true`

----------

#### object.extend(module) ####

向模块中添加单体方法，然后附加到对象上。

----------

#### object.hash() ####

返回对象的哈希值，当存储键时使用 **Hash** 模块实现。默认时，为每个对象返回一个唯一的16进制数字。如果两个对象通过 `equals()` 方法进行比较，两者必须返回相同的哈希值，否则不能在 **Hash** 中正确执行。

----------

#### object.isA(type) ####

如果对象符合给定类型的实例时返回真值，给定的类型应该为类或模块。如果给定的类型处于对象继承链上的任意位置时则都将返回真值。

----------

#### object.method(name) ####

返回对象中指定方法的一个副本，并且作为一个独立函数被绑定到对象实例变量上。关于绑定可以查看[这里](./binding.md)。

----------

#### object.tap(block, context) ####

在给定的上下文环境中调用函数块，将对象自身作为参数传递给函数块，最后返回对象自身。常应用于在很长的方法链中打断点检验输出是否正确，利于调试之用。例如：

	list                 .tap(function(x) { console.log("original: ", x) })
	  .toArray()         .tap(function(x) { console.log("array: ", x) })
	  .select(condition) .tap(function(x) { console.log("evens: ", x) })
	  .map(square)       .tap(function(x) { console.log("squares: ", x) });