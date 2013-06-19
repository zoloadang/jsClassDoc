# Debugging support -- 调试 #

### 概述 ###

在新发布的2.I版本中介绍到，已经支持了 `WebKit的displayName`属性进行性能测试和调试JS。基本上，它是一种针对JS对象和方法未经过命名而直接定义，可以被赋值给任意数量的变量，造成对象和函数多为匿名的解决尝试。

WebKit的性能检测和调试器利用给赋值的函数对象添加 `displayName` 属性来改善了此种形式。JS.Class为方法和内部类生成了显示名称，只需在类的最外层指定一个名称即可。类(包括模块)名称是可选项而且作为类和模块构造函数的首位参数。例如：

	Foo = new JS.Module('Foo', {
	    sleep: function() { /* ... */ },
	
	    extend: {
	        eatFood: function() { /* ... */ },
	
	        InnerClass: new JS.Class({
	            haveVisions: function() { /* ... */ }
	        })
	    }
	});

> 该显示名并不强制要求和赋值的变量名一致，尽管使用同样的名称可能会更有帮助。这个名称不能用于变量赋值。

按照上面代码中的定义，我们可以在方法和内部类中查找到 **displayName** 设置：

	Foo.instanceMethod('sleep').displayName
	// -> "Foo#sleep" 
	
	Foo.eatFood.displayName
	// -> "Foo.eatFood" 
	
	Foo.InnerClass.displayName
	// -> "Foo.InnerClass" 
	
	Foo.InnerClass.instanceMethod('haveVisions').displayName
	// -> "Foo.InnerClass#haveVisions"

> 【扩展知识】 WebKit团队在这个问题采取了有点儿另类的策略。囿于函数（包括匿名和命名函数）如此之差的表现力，WebKit引入了一个“特殊的” `displayName`属性（本质上是一个字符串），如果开发人员为函数的这个属性赋值，则该属性的值将在调试器或性能分析器中被显示在函数“名称”的位置上。

[命名函数表达式探秘译文推荐](http://www.jb51.net/onlineread/named-function-expressions-demystified/#webkit-displayName)