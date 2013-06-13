# Singleton methods -- 单例方法 #

### 概述 ###

在Ruby中，单例方法是通过将方法附加到单例对象上而不是通过类来定义的。如果你已经为Ruby对象添加了一个单例方法，那么 `super`的引用是对象所属的类定义。JS.Class也继承了同样的行为。再次调用我们前面创建的**Animal**类：

	var Animal = new JS.Class({
	    initialize: function(name) {
	        this.name = name;
	    },
	    speak: function(things) {
	        return 'My name is ' + this.name + ' and I like ' + things;
	    }
	});

我们可以创建一个新动物，然后使用单例方法进行扩展。所有通过JS.Class派生的对象都会默认带有`extend()`扩展方法：

	var cow = new Animal('Daisy');
	
	cow.extend({
	    speak: function(stuff) {
	        return 'Mooo! ' + this.callSuper();
	    },
	    getName: function() {
	        return this.name;
	    }
	});
	
	cow.getName()   // -> "Daisy" 
	
	cow.speak('grass')
    // -> "Mooo! My name is Daisy and I like grass"


----------

### 通过模块进行对象扩展 ###

不仅可以通过传递简单对象给`extend()`方法，而且也可以通过模块来扩展。接收对象会将模块中的所有方法进行扩展。例如我们如果想扩展**Observable**模块的话：

	cow.extend(JS.Observable);
	
	cow.addObserver(function() {
	    alert('This cow is observable!');
	});
	
	cow.notifyObservers();

如预期所料一样，弹出提示框“This cow is observable!”。使用模块进行对象扩展存在一些很有意思的继承关系，具体信息可以查看[继承](./Inheritance.md)这一章内容。简而言之，所有存储在模块中的单例方法都会被附加到对象上 -- 在Ruby领域常称这个对象为`eigenclass`或`metaclass`。通过模块来扩展对象时，则该模块会被直接混入到`eigenclass`中，成为继承树中的一部分。所以，我们可以重写`notifyObservers()`方法，例如复制两遍观察者，调用**callSuper()**将会尝试访问通过模块扩展后的对象。

	cow.extend({
	    notifyObservers: function() {
	        this.callSuper();
	        this.callSuper();
	    }
	});
	
	// alerts "This cow is observable!" twice
	cow.notifyObservers();
