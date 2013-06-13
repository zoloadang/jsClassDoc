# Inheritance - 继承 #

----------

### 概述 ###

要弄明白Ruby的继承机制，必须要明白它是如何查找方法的。来自于《The Ruby Programming Language》和Ruby之父松本行弘的解释如下：

> 当Ruby执行一个方法调用时，它首先找出被调用的方法。这个操作过程被称为方法寻找或方法名解析。例如方法调用表达式： **o.m **，Ruby会按照如下步骤进行解析：

> 1. 首先，它检验 **o **的eigenclass中是否存在单例方法 **m **.

> 2. 如果在eigenclass中没有找到m，那么Ruby就继续搜索o的类中是否存有名称为m的实例方法。

> 3. 如果在类中没有方法m，Ruby就继续在o的类中包含的任意模块中搜索实例方法。当类中包含有多个模块情况下，则按照包含进来的顺序反向搜索，也就是说最后包含进来的模块首先被搜索。

> 4. 如果类中或类中包含的模块中均未找到方法m，那么根据继承关系向上至超类中按照步骤2和3重复搜索，直到在每个祖先类或其包含模块中找到符合的方法为止。

听起来是不是很迷糊？没关系，接下来让我们通过一些实例来解释。

### 父子类继承关系 ###


我们从基于类的继承开始讲起，因为这是最广泛普遍的理解。如果 **Child **类继承于 **Parent **类，那么 **Child **类的方法可以通过 **callSuper() **来调用 **Parent **类中的同名方法。 **Child **可以继承 **Parent **类中的全部方法甚至在必要时可以进行重写操作。

	var Parent = new JS.Class({
	    speak: function() {
	        return "I'm an object";
	    },
	
	    writeCode: function(code) {
	        return "I wrote: " + code;
	    }
	});
	
	var Child = new JS.Class(Parent, {
	    speak: function() {
	        return this.callSuper().replace(/[aeiou]/ig, '_');
	    }
	});
	
	(new Child).speak()
	// -> "_'m _n _bj_ct" 
	
	(new Child).writeCode('function(){}')
	// -> "I wrote: function(){}"

> 这种继承类型在面向对象语言中都是普遍支持的而且容易理解：一个类只能有一个父类，同时可以调用父类中的所有方法。值得一提的是，和Ruby一样，JS.Class中 **callSuper() **方法的参数也是可选的。默认情况下，会将子类方法参数完整地传递到父类方法中。更多信息可以查看 **[Creating classes](creating_classes.md) **

### 混入继承 ###

这块内容容易引起理解上的困惑，因为它涉及到的是多继承，但是Ruby的方法查找规则可以预知依赖结果。当给一个类混入模块时，这个模块就变成了继承树的一部分。Ruby查找规则会在类中包含的模块中进行搜索(按载入逆序查找，或称为深度优先搜索)，如果没有找到目标才进入父级类继续查找。如果一个模块中再包含其它多个模块时，那么在继续前进之前先要将这些内嵌模块搜索一遍。下面通过一个例子应该可以理解的更清楚一些：

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

假设我们创建一个实例对象 **new Foo() **并且调用 **speak() **方法。在类Foo中包含了模块ModA和ModC，然后在ModC中又载入了ModB。所以当在Foo中寻找方法时，顺序为ModC，ModB，ModA。在每个阶段，`callSuper()`方法都会迫使在继承链中不断搜索。

	(new Foo).speak()
	// -> "speak() in ModA, speak() in ModB, and in ModC, and in class Foo"

这个例子很好地阐述了`callSuper`的晚绑定的使用。`ModB`中不包含其它模块，那么当它内部调用`callSuper`方法时，依赖于`ModB`内部引起的模块和在继承链中的其它模块。所以上面的例子中ModB调用callSuper时，在继承链的ModA模块中找到了speak方法。

> 谨记：当搜索方法时进入父级类之前会搜索所有子类中包含的模块。

### 晚绑定 ###

类的继承链可以通过混入更多模块的方式任意修改。看下上面例子的扩展：

	var ModD = new JS.Module({
	    speak: function() {
	        return 'ModD speaks';
	    }
	});
	
	Foo.include(ModD);

> 现在`Foo`的查找链顺序变为Foo->ModD->ModC->ModB->ModA；然而在ModD模块中的`speak()`方法中未调用`callSuper()`，所以在ModC之前的方法将不会被调用。

	(new Foo).speak()
	// -> "ModD speaks, and in class Foo"

### 单例方法 ###

当我们通过类来创建一个对象时，该对象不存在属于其自己的方法，它包含的所有方法都继承自它的类。

*Example:*

	var Machine = new JS.Class({
	    run: function() {
	        return 'Machine is running';
	    }
	});
	
	var obj = new Machine();
	obj.run()   // -> "Machine is running"

然而，我们可以给这种通过类生成的对象扩展属于它们自己的方法。每一个这样的对象都有一个`eigenclass`(在一些文献中也不称作元类`metaclass`)，当给对象添加方法时则会被存在这个`eigenclass`中。直接对象中生成的方法(除了在类中或模块中定义的方法)被称为单例方法，因为它们是在单独对象上直接定义的。当调用对象上的方法时JS.Class层面上的类是处于首位的。

	obj.extend({
	    run: function() {
	        return this.callSuper() + ', and obj is running';
	    }
	});

`callSuper`会通向哪里呢？当我们在对象的`eigenclass`中寻找完后，再继续到对象所属的类中搜寻，在上面这个例子中就是指 **Machine **类。所以，我们会得到：

	obj.run()
	// -> "Machine is running, and obj is running"

但是还要啰嗦一遍，记住在进入其父类寻找之前总是会搜索所有其包含的模块。所以如果我们使用模块来进行扩展一个对象时，那么该模块会被载入到对象的`eigenclass`中，并且搜索顺序会介于对象和其所属类之间：

	var MachineExtension = new JS.Module({
	    run: function() {
	        return this.callSuper().toUpperCase();
	    }
	});
	
	obj.extend(MachineExtension);

上面的代码例子中不能快速地明白`callSuper()`要做什么：因为 **MachineExtension **既没有父类也没有载入任何的模块。由于Ruby的方法寻找规则是晚绑定的，也就是说 **MachineExtension **中的`callSuper()`依赖于被使用的上下文。针对上段代码中最后一行你可以理解成等价于：

	obj.__eigen__().include(MachineExtension);

那么，在对象的`eigenclass`中载入了模块 **MachineExtension **，也就是令其成为了继承树的一部分。所以当调用`obj.run()`时，调用栈的顺序应该为obj.run -> MachineExtension#run -> Machine#run。相应的 **Machine#run **返回了字符串“Machine is running”，而MachineExtension#run返回了大写，最后obj.run添加了字符串“, and obj is running”。

	obj.run()
	// -> "MACHINE IS RUNNING, and obj is running"

呼~，确实不怎么简单啊！这完全属于Ruby的高级功能，但是如果你能牢牢记住如下几条规则你就能很好地把握它：寻找一个方式，首先在对象的`eigenclass`中寻找，接着再到对象所属的类中继续找，然后再继续深入到所属类的父类中找等等。在每个阶段中，深入父级前要找遍所有载入的模块中是否存在要找的方法。