# Class methods -- 类方法 #

### 概述 ###

在Ruby中模块和类被定义成另一种类型的对象；它们是可以存储方法和生成新对象的对象。类方法被誉为一种特殊的类对象，但不是类的实例对象。它们其实是[单例方法](./singleton_methods.md)的一种特殊体。当定义类时可以使用`extend`语句块来添加类方法：

	var User = new JS.Class({
	    extend: {
	        find: function(id) {
	            // Return a User with id
	        },
	        create: function(name) {
	            return new this(name);
	        }
	    },
	    initialize: function(name) {
	        this.username = name;
	    }
	});

而且，也可以在创建完类之后再添加方法：

	var User = new JS.Class({
	    initialize: function(name) {
	        this.username = name;
	    }
	});
	
	User.extend({
	    find: function(id) {
	        // Return a User with id
	    },
	    create: function(name) {
	        return new this(name);
	    }
	});

上面这两种语法同样适用于创建和扩展[模块](./using_modules.md)。在类中的方法里使用的关键字`this`引用的是类本身 -- 可以通过**User.create()**方法来验证：

	var james = User.create('James');
	james.username    // -> 'James'
	james.klass       // -> User

当创建了子类的时候，其会继承父类的任意类方法，而且可以通过**callSuper()**方法访问：

	var LoudUser = new JS.Class(User, {
	    extend: {
	        create: function(name) {
	            return this.callSuper(name.toUpperCase());
	        }
	    }
	});
	
	var me = LoudUser.create('James');
	me.username   // -> 'JAMES'
	me.klass      // -> LoudUser
	
	var you = LoudUser.find(24)   // inherited from User

需要注意的是子类中的 `this`，甚至使用 `callSuper`方法时，它的作用域永远都是指方法调用的地方。比如上面代码中我们返回的是**LoudUser**而不是**User**。

另外注意一下，这里的类方法不同于`Java`中的静态方法；如果想要通过类实例来调用一个类方法时，必须要先通过实例对象的`klass`属性得到类引用。

	User.define('copy', function() {
	    return this.klass.create(this.username);
	});