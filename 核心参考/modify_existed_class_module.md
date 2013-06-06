# Modifying existing classes and modules #

----------

### 概述 ###

正如在Ruby中的类和模块可以在任何时候打开进行修改一样，你可以在已存在的类中添加和修改方法。在JS.Class中所有的类和模块都带有一些特殊的方法可以对其包含的方法进行修改。接下来介绍的是针对修改类的，但同样适用于修改模块，因为类是可以进行初始化的模块。

首先介绍的第一个方法是`define()`；这个方法的作用是添加一个单独命名函数到类/模块中。如果你正在使用该方法修改一个类，那么添加的方法会在类或子类实例中立刻编程可用的状态。

	Animal.define('sleep', function() {
	    return this.name + ' is sleeping';
	});
	
	rex.sleep()
	// -> "Rex is sleeping"

> 注意，当你打算添加的方法与类中已存在的方法同名时，则类中的原方法会被覆盖掉。在父类中的和被混入进来的同名方法可以通过**callSuper()**进行访问。

如果想要给已经存在的方法创建别名时，可以使用`alias()`方法。规则就是新的别名作为键，而旧的方法名作为值放在右侧。

    Animal.alias({
	    talk:   'speak',
	    rest:   'sleep'
	});

另外还有一个可用的方法`include()`。这个方法担任着多个角色；如果你提供的是一个模块，那么该模块将被混入到类中同时成为继承树的一部分。然而如果提供的是对象类型，则该对象中的方法直接被复制到类自身，若类中存在同名方式时会被覆盖掉。

	// Replace Dog's speak method  (#1)
	Dog.include({
	    speak: function(stuff) {
	        return this.callSuper('lots of ' + stuff) + '!';
	    }
	});
	
	rex.speak('cats')
	// -> "My name is Rex and I like lots of cats!" 
	
	// Mix in a module, altering the class's ancestry
	// callSuper() in Dog#speak will now call this method
	var Speaker = new JS.Module({
	    speak: function(stuff) {
	        return 'I can talk about ' + stuff + '!';
	    }
	});
	
	Dog.include(Speaker);
	rex.speak('cats')
	// -> "I can talk about lots of cats!!"

注意一下，在上面这段代码中所示，包含的**Speaker**并未将`Dog`类中的`speak`方法(代码中标记 `#1` 的地方)覆盖掉。在Dog类中定义的方法只有通过直接在Dog类中重新定义才可以被覆盖。`Speaker`只是将另一个同叫`speak`的方法掺入到`Dog`类的继承链中而已。需要了解更多相关信息，可以阅读[这篇文档](./Inheritance.md)。