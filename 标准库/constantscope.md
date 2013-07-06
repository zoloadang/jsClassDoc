# ConstantScope - 常量作用域 #

### 概述 ###

**ConstantScope** 是一种元编程掺入类，它改变了常量只用来在模块和类中作为存储的形式。在包含它的类中不会添加任意方法，但是可以通过一些方式来改变它自身的结构。

	// In the browser
	JS.require('JS.ConstantScope', function(ConstantScope) { ... });
	
	// In CommonJS
	var ConstantScope = require('jsclass/src/constant_scope').ConstantScope;

在JavaScript语言中的根本没有常量这种概念除了规范约定，约定任何变量名首字母是大写的都会被称作常量。在Ruby中也是类似的规范，不同的是如果想重定义一个常量时解释器会报警告提示。要弄清楚这个模块的用途，我们首先要观察一下Ruby和jsclass之间对于常量查找的不同点。

jsclass始终在尝试模拟并提供和Ruby近似的对象继承系统，比如方法查找在两个系统中完全一致，然而对于常量查找就不一样了。因为Ruby中的常量查找系统采用的是常量命名的词法作用域，如果想能正常地工作需要一个语言解析器来支持。而jsclass并不是代码解析器，也就是说jsclass不支持与Ruby一样的常量语法。让我们通过具体的实例来说明：

	class Outer
	  CONST = 45
	
	  class Item          # (1)
	  end
	
	  class Inner
	    class Item        # (2)
	      def initialize
	        puts CONST    # (3)
	      end
	    end
	
	    def create_item
	      Item.new        # (4)
	    end
	  end
	
	  def create_item
	    Item.new          # (5)
	  end
	end 

> 在Ruby中，任意首字母以大写形式存在的名称都被认为是一个常量：类 `Outer` 包含了三个常量： `CONST, Item 和 Inner` 。它们分别引用的是 `Outer::CONST, Outer::Item 和 Outer::Inner` 。同样地， `Outer::Inner` 中包含了一个常量， `Outer::Inner::Item` 。当引用到一个常量时，Ruby会在词法作用域中进行查找，比如，在封装的类中寻找，然后逐层向上继续，直到全局作用域。所以，在标记(4)的引用的是 `Outer::Inner::Item` (在标记(2)处定义的)，标记(5)引用的是 `Outer:Item` (在标记(1)处定义)。最后标记(3)不得不回到 `Outer` 才能发现常量 `CONST` 。


如果使用jsclass来实现同样目的的话，我们需要添加类的常量属性得通过 `extend` 来扩展：

	Outer = new Class({
	    extend: {
	        CONST: 45,
	
	        Item: new Class(),
	
	        Inner: new Class({
	            extend: {
	                Item: new Class({
	                    initialize: function() {
	                        alert(Outer.CONST);
	                    }
	                })
	            },
	
	            create_item: function() {
	                return new this.klass.Item();
	            }
	        })
	    },
	
	    create_item: function() {
	        return new this.klass.Item();
	    }
	}); 

对比下来，不难发现使用jsclass的方式过于繁杂，使用了太多的内嵌。由于jsclass模拟的类不能支持词法嵌套，所以使用过多的名称重复(Outer.CONST)等，而且还有看起来挺好笑的 `this.klass.X` 这样的引用，因为实例方法只有通过这种方式才能访问到类中存储的常量值。

这时 **ConstantScope** 粉墨登场了，它是一个模拟Ruby的常量寻找系统的模块，其可以在类及其任意内嵌类中进行查找，要寻找任意常量(首字母大写的属性)都可以通过简单地使用 `this.X` 方式来实现。获得引用值的词法规则与Ruby的类似，所以我们再重新编写一下上面的代码：

	Outer = new Class({
	    include: ConstantScope,
	
	    CONST: 45,
	
	    Item: new Class(),
	
	    Inner: new Class({
	        Item: new Class({
	            initialize: function() {
	                alert(this.CONST);
	            }
	        }),
	
	        create_item: function() {
	            return new this.Item();
	        }
	    }),
	
	    create_item: function() {
	        return new this.Item();
	    }
	}); 

> 我们可以注意到在上面的代码中我们清除了额外的 `Outer` 引用去查找 `CONST`的过程，没有 **extend** 语句块，而且不再需要任何的 `this.klass.X` 这样的引用了。同时意味着在类和实例方法中拥有了一样的引用常量的语法支持：

	SomeClass = new Class({
	    include: ConstantScope,
	
	    MY_CONST: 'cheese',
	
	    fetch: function() {
	        return this.MY_CONST;
	    },
	
	    extend: {
	        get: function() {
	            return this.MY_CONST;
	        }
	    }
	});
	
	SomeClass.get();      // -> "cheese" 
	
	var s = new SomeClass();
	s.fetch();            // -> "cheese"


### 注意！ ###

JavaScript是很简单的，它无法支持和Ruby一样的常量语法，因为Ruby无需面对一大堆嘈杂烦人的全局变量。为了在所有嵌套类和方法中支持 `this.X` 这种语法，**ConstantScope** 模块需要处理一大堆的反射和创建大量的额外模块来确保常量能够在嵌套的类中可以正确继承，同时可以在类和实例方法中均可正常使用。所有这些工作耗费是巨大的，所以可能会遇到性能瓶颈问题；目前该模块还处于实验阶段，所以要想使用它需要注意以下提到的。

该模块通过在大量的不同对象之间传播常量来进行工作的，所以类似于处在词法作用域中。当使用一个简单属性存取器来修改常量时，不能自动将新值进行传播的，要确保可以正常工作需要使用 `extend` 来重新赋值常量。拿上面例子来说明：

	// This will not propagate as expected
	SomeClass.MY_CONST = 'cake';
	
	// Do this instead
	SomeClass.extend({MY_CONST: 'cake'});