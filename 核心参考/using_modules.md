# Using Modules -- 使用模块 #

----------

### 概述 ###

在Ruby中模块也是一种简单的对象，它可以用来存储方法。而在JS.Class中它是类库的核心成员，被应用于操作继承树的规则，寻找指定方法等等。 **JS.Module **和 **JS.Class **都是类，但是后者继承自前者。也就是：

    JS.Class.superclass === JS.Module

我喜欢把类理解成模块的一种而且是可以初始化创建对象的。但是理论上重要的说明是：模块用来保存和组织整理方法，通过混入的方式应用到类中。

例如，假设你想要把对象之间通过比较来进行排序的方法封装起来。(JS.Class已经带有了一个叫做`Comparable`的模块，所以就无需自己动手重新编写了)想象中的模块应该看起来是这样的：

	var Comparable = new JS.Module({
	    lt: function(object) {
	        return this.compareTo(object) == -1;
	    },
	    lte: function(object) {
	        return this.compareTo(object) < 1;
	    },
	    gt: function(object) {
	        return this.compareTo(object) == 1;
	    },
	    gte: function(object) {
	        return this.compareTo(object) > -1;
	    },
	    eq: function(object) {
	        return this.compareTo(object) == 0;
	    }
	});

> 这个 **Comparable **模块无法进行初始化 -- 模块不能用来生成新对象。但是可以将模块添加到类中，然后该类可以得到模块中存储的全部方法。这个模块可以和任意类中的`compareTo()`方法一起工作：

	var User = new JS.Class({
	    include: Comparable,
	
	    initialize: function(name) {
	        this.name = name;
	    },
	
	    compareTo: function(user) {
	        if (this.name < user.name)
	            return -1;
	        else if (this.name > user.name)
	            return 1;
	        else
	            return 0;
	    }
	});

哇哦，现在可以使用比较方法了：

	var jack = new User('Jack'), jill = new User('Jill');
	jack.lt(jill)   // -> true
	jack.gt(jill)   // -> false
	jill.gt(jack)   // -> true

> 另外需要注意的一点，存储在模块中的方法并不是模块的方法，也就是说不能通过模块直接调用存储的方法，例如 `Comparable.lt(someObject)` 这样是失败的。这些存储在模块中的方法只能通过包含该模块的类创建的对象进行调用。

如果你想要在类中载入多个模块，那么必须要通过指定一个数组来实现，载入的顺序可以自己控制，例如：

	var Foo = new JS.Class({
	    include: [ModA, ModB, ModC],
	
	    initialize: function() {
	        // ...
	    }
	});

或者，你可以先创建类，然后再混入模块：

	var Foo = new JS.Class({
	    initialize: function() {
	        // ...
	    }
	});
	Foo.include(ModA);
	Foo.include(ModB);
	Foo.include(ModC);

### 模块 与 callSuper() ###

当你在类中引入一个模块时，该模块瞬间就会成为类祖先的一部分。也就是意味着在类中可以通过使用`callSuper()`方法来重写继承自模块中的同名方法。例如，让我们尝试着重写一下 **Comparable#eq() **方法，然后记录其结果：

	User.define('eq', function(user) {
	    var areEqual = this.callSuper();
	    if (areEqual) console.log("Found two equal objects!");
	    return areEqual;
	});

注意，我们不需要将`user`参数显式传给`callSuper()`方法，如果我们不要求重载参数的话，`user`参数会被自动传入进去。需要了解更多继承相关信息，可以[看这里](./Inheritance.md)