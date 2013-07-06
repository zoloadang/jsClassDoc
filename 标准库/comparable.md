# Comparable - 比较 #

### 概述 ###

**Comparable** 是一个用来帮助操作对象之间排序问题的模块。它借鉴于Ruby中的Comparable，只是没有像Ruby中所用的漂亮的方法名，如 `<=` 等等。

	// In the browser
	JS.require('JS.Comparable', function(Comparable) { ... });
	
	// In CommonJS
	var Comparable = require('jsclass/src/comparable').Comparable;

当使用该模块时，在类中必须要定义一个实例方法 **compareTo**，用来说明与其它对象的比较关系。例如，假设创建一个类来描述待办事件列表项。可以通过事件优先级来排序。

	var TodoItem = new Class({
	    include: Comparable,
	
	    initialize: function(task, priority) {
	        this.task = task;
	        this.priority = priority;
	    },
	
	    // Must return -1 if this object is 'less than' other, +1 if it is
	    // 'greater than' other, or 0 if they are equal
	    compareTo: function(other) {
	        if (this.priority < other.priority) return -1;
	        if (this.priority > other.priority) return 1;
	        return 0;
	    }
	});

`TodoItem` 含有如下实例方法：

 - `lt(other)` -- 当接收者小于other时返回true
 - `lte(other)` -- 当接收者小于或等于other时返回true
 - `gt(other)` -- 当接收者大于other时返回true
 - `gte(other)` -- 当接收者大于或等于other时返回true
 - `eq(other)` -- 当接收者等于other时返回true
 - `between(a,b)` -- 当接收者包含在 a 和 b 区间内时返回true
 - `compareTo(other)` -- 通过返回的 `-1/0/1` 来指示比较结果

`TodoItem` 还有一个类方法 -- **compare** 用来排序。

接下来让我们创建一些待办事件项，然后看下效果：

	var items = [
	    new TodoItem('Go to work', 5),
	    new TodoItem('Buy milk', 3),
	    new TodoItem('Pay the rent', 2),
	    new TodoItem('Write code', 1),
	    new TodoItem('Head down the pub', 4)
	];
	
	items[2].lt(items[1])   // -> true
	items[0].gte(items[3])  // -> true
	items[4].eq(items[4])   // -> true
	items[1].between(items[3], items[4])  // -> true
	
	items.sort(TodoItem.compare)
	// -> [
	//        {task: 'Write code', ...},
	//        {task: 'Pay the rent', ...},
	//        {task: 'Buy milk', ...},
	//        {task: 'Head down the pub', ...},
	//        {task: 'Go to work', ...}
	//    ]