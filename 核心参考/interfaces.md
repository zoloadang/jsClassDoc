# Interfaces -- 接口 #

### 概述 ###

尽管在Ruby中没有发现接口的使用，但还是决定在JS.Class中进行支持。接口的概念出自于Java语言，若灵活应用的话在JavaScript中会起到很好的作用。接口的原理是创建一批没有具体实现只有方法名的对象。然后可以通过具体对象或类来对接口方法进行实现；当需要的对象使用了接口中定义的方法时，如果方法没有具体实现则会抛出异常。

如果想要创建一个接口，需要做的就是传入一个含有方法名的数组：

	var IntComparable = new JS.Interface([
	    'compareTo', 'lt', 'lte', 'gt', 'gte', 'eq'
	]);
	
	var IntStateMachine = new JS.Interface([
	    'getInitialState', 'changeState'
	]);

接下来，就可以测试任意对象是否实现了提供的接口中的所有方法：

	JS.Interface.ensure(someObject, IntComparable, IntStateMachine);

`JS.Interface.ensure` 检测提供的首位参数对象内是否全部实现了接口参数中方法。如果有一个测试失败，就会立即抛出错误提示对象中没有实现的方法名称。