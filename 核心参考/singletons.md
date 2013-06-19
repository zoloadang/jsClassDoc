# Singletons -- 单体 #

### 概述 ###

一个单体类只能生成一个实例。这个概念在Java语言中非常有用，必须要先创建类才能再生成对象。JavaScript中不通过类来生成对象(只有对象，没有类)，但是可以通过 **JS.Singleton** 借助存在的 **JS.Class** 创建的类来生成自定义对象，可以继承方法，包括模块和 `method()` 方法等等。

	//Animal在前面例子中创建的类
	var Camel = new JS.Singleton(Animal, {
	    fillHumpsWithWater: function() { ... }
	});
	
	// You can call instance methods...
	Camel.speak('the desert');    // from Animal
	Camel.fillHumpsWithWater();
	
	var s = Camel.method('speak');
	s('drinking');
	
	Camel.klass.superclass    // -> Animal

> `JS.Singleton` 通过给定的参数创建了一个类，且立即实例化并返回该实例对象。像上面代码中所示，可以通过 `Camel.klass` 来访问所属类。