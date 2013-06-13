# Metaprogramming hooks -- 钩子方法 #

### 概述 ###

在Ruby中当检测一个类是否为子类或模块是否被混入时，会定义一些钩子方法来实现。这些钩子方法常被称为 `inherited()`，`included()` 和 `extended()`。

如果一个类中定义了一个叫做`inherited()`的类方法时，每当你创建子类时就会调用该类方法：

	var ChildDetector = new JS.Class({
	    extend: {
	        inherited: function(klass) {
	            // Do stuff with child class
	        }
	    }
	});

这个钩子方法接收到一个新的子类作为参数。注意`class`在JavaScript中属于保留字，所以不能作为变量名来使用。当 **inherited() **方法被调用后，子类中的所有方法均可以在回调函数中正常使用。

同样的情况，如果使用`include()`方法载入一个模块时，会直接调用在类中定义的 **included **类方法。这样就可以针对不同的模块重新定义载入的意义了。

另外的`extended()`钩子方法类同于 `included()`方法，只是其调用时机是当通过 **extend() **方法将模块扩展到对象时。

	// This will call MyMod.extended(obj)
	obj.extend(MyMod);

同样地，可以使用`extended`钩子来针对不同模块进行重定义，以达到改变扩展对象的行为动作。