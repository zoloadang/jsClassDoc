# Method Binding -- 方法绑定 #

### 概述 ###

在Ruby中使用方法对象(`Method`和`UnboundMethod`)为某个对象方法传递引用，因此可以不使用对象引用来直接调用一个方法。JavaScript中的函数被称为一等公民，所以也可以通过引用一个方法然后再调用它：

	var rex = new Dog('Rex');
	var spk = rex.speak;    // 一个引用但并未调用执行方法
	spk('biscuits');
    // -> "MY NAME IS AND I LIKE BISCUITS!"

注意到Rex的名字不知道去哪里了？原因是，我们没有通过对象rex来调用spk方法，所以在方法内部的`this`关键字没有引用到正确的对象。而JS.Class为每个对象提供了一个`method()`方法，其通过名称返回一个方法且绑定到它正确的源对象上。这个方法是一个简单的JavaScript函数，通过该方法便可以独立调用并且不用担心`this`的上下文环境了：

	var speak = rex.method('speak');
	speak('biscuits');
    // -> "MY NAME IS REX AND I LIKE BISCUITS!"

同理，也适用于类方法，因为类也是对象：

	var User = new JS.Class({
	    extend: {
	        create: function(name) {
	            return new this(name);
	        }
	    },
	    initialize: function(name) {
	        this.username = name;
	    }
	});
	
	var u = User.method('create');
	u('James')    // -> {username: 'James'}