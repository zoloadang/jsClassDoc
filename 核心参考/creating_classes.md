# 创建类 #

----------

在很多面向对象系统中类是构建模块的基础。JavaScript(亦称ECMAScript3)各个版本中均不支持类，尽管在一些语法中原型链`prototype`对象和构造函数看起来很像类。在JavaScript中创建基于类的编程还是比较麻烦的，但是`JS.Class`可以将其简单化。

生成一个类，只需要通过  **new JS.Class()** 即可，类的方法相当于JavaScript的常规函数。

	var Animal = new JS.Class({
	    initialize: function(name) {
	        this.name = name;
	    },
	
	    speak: function(things) {
	        return 'My name is ' + this.name + ' and I like ' + things;
	    }
	});

> 通过上面的代码段，我们可以发现在类中含有一个`initialize()`初始化方法，当构造实例对象时没有参数的话，这个初始化方法可以省略。当生成实例时`initialize()`方法被调用，并且参数被应用与新生成的实例对象。

	var nemo = new Animal('Nemo');    // nemo.name == "Nemo" 
	
	nemo.speak('swimming')
	// -> "My name is Nemo and I like swimming"

## 继承父类 ##

接下来，让我们模拟更多动物的种类。假设创建一只狗：

	var Dog = new JS.Class(Animal, {
	    speak: function(stuff) {
	        return this.callSuper().toUpperCase() + '!';
	    },
	
	    huntForBones: function(garden) {
	        // ...
	    }
	});

>  在Dog类中不需要`initialize()`方法，因为它继承于 **Animal **类。然而，它选择重载了`speak()`方法。

现在我们注意到使用`JS.Class`生成了一个特殊的方法 --  **callSuper()**。这个方法在类方法中动态创建并可以通过它访问到父类中的同名方法。类似于Ruby，不用强制给callSuper()传递参数，因此避免了很多重复性动作。当前子类方法的参数会通过callSuper()自动传递给父类同名方法。

	var rex = new Dog('Rex');
	rex.speak('barking')
	// -> "MY NAME IS REX AND I LIKE BARKING!"

 **注意： ** `callSuper()`不能在对象外直接访问：

	rex.callSuper();
	// -> rex.callSuper is not a function

如果将参数直接传递给`callSuper()`方法时，则会自动被重载掉相应个数的参数，例如：

	var Dog = new JS.Class(Animal, {
	    speak: function(stuff) {
	        stuff = stuff.replace(/[aeiou]/ig, '_');
	        return this.callSuper(stuff).toUpperCase() + '!';
	    }
	});
	
	var rex = new Dog('rex');
	rex.speak('something')
	// -> "MY NAME IS REX AND I LIKE S_M_TH_NG!"

> 当重载参数时应多加注意，要明白`callSuper()`总是会将当前方法的所有参数进行传递的，除了指定要重载的参数外。所以，假设你的方法含有A,B,C,D和E四个参数：

	this.callSuper('one', 'two')

> 那么实际传递的参数应该如下所示：

	this.callSuper('one', 'two', C, D, E)

相比于简单地调用父类方法，Ruby的继承系统要强大很多倍。需要查看更多继承方面的信息，可以点[这里](./inheritance.md "更多继承相关信息")。
