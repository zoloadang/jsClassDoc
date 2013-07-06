# Enumerable - 可枚举 #

### 概述 ###

**Enumerable** 模块基本上是直接将Ruby的Enumerable模块转成JavaScript。只是部分方法因为坚持JavaScript规范命名稍有不同，但总体底层理念都是一样的：该枚举模块提供的方法都被用在类中进行集合或列表的展现。使用模块的约定是在类中要通过 `forEach` 方法来遍历集合中的每一个成员。当使用 `forEach` 方式时若未提供迭代函数的话，则应该返回一个 `Enumerator` (枚举器) -- 看下面的例子。

	 // In the browser
	JS.require('JS.Enumerable', function(Enumerable) { ... });
	
	// In CommonJS
	var Enumerable = require('jsclass/src/enumerable').Enumerable;

下面是一个简单的例子，在一个类中将它的实例部分数据存储到列表中。类可能会选择任何方式来存储集合，而且不必一定要保证某个特定迭代次序； `forEach` 方法的目的是用来封装存储机制的，方便使用类的用户。

	var Collection = new Class({
	    include: Enumerable,
	
	    initialize: function() {
	        this._list = [];
	        for (var i = 0, n = arguments.length; i < n; i++)
	            this._list.push(arguments[i]);
	    },
	
	    forEach: function(block, context) {
	        if (!block) return this.enumFor('forEach');
	
	        for (var i = 0, n = this._list.length; i < n; i++)
	            block.call(context, this._list[i]);
	
	        return this;
	    }
	});

创建一个实例然后看下结果：

	var list = new Collection(3,7,4,8,2);
	list.forEach(function(x) {
	    console.log(x);
	});
	
	// prints:
	//    3,  7,  4,  8,  2

下面会逐一介绍 **Enumerable** 模块提供给 `Collection` 类的API。在每个方法的参数列表中，`block` 是一个函数而 `context` 是可选参数指的是 `block` 所属的上下文作用域。在很多方法中，`block` 可能是一个字符串，类似于Ruby中的 `Symbol#to_proc` 功能。如下所示：

	var strings = new Collection('iguana', 'labrador', 'albatross');
	
	strings.map('length')
	// -> [6, 8, 9]
	
	strings.map('toUpperCase')
	// -> ["IGUANA", "LABRADOR", "ALBATROSS"]

字符串可能指代的是一个方法或集合中对象的属性，作为首位参数的命名方法调用时会被转换为一个函数，然后将剩余的参数作为被转换后的方法的参数执行。例如：

	strings.map('toUpperCase')
	// is converted to:
	
	strings.map(function(a, b, ...) { return a.toUpperCase(b, ...) })

大部分的JavaScript二元操作符都是被支持的，所以可以循环使用 `inject` 操作，例如：

	new Collection(1,2,3,4).inject('+')
	// -> 10
	
	var tree = {A: {B: {C: 87}}};
	new Collection('A','B','C').inject(tree, '[]')
	// -> 87

#### all(block, content) ####

当集合中所有元素被block执行后皆返回true，那么all则返回true。如果没有提供block，而同时集合中的每个元素都是真值时也同样返回true。它的别名为 **every()** 。

	var list = new Collection(3,7,4,8,2);
	
	list.all(function(x) { return x > 5 });
	// -> false
	
	list.all(function(x) { return typeof x == 'number' });
	// -> true
	
	new Collection(3,0,5).all();
	// -> false

#### any(block, context) ####

当集合中每个元素被block执行时有一个或多个返回true则为真值。同理，当未提供执行函数时，若集合中任意元素为真就返回true。别名为 **some()** 。

	var list = new Collection(3,7,4,8,2);
	
	list.any(function(x) { return x > 5 });
	// -> true
	
	list.any(function(x) { return typeof x == 'object' });
	// -> false
	
	list.any();
	// -> true
	
	new Collection(0, false, null).any();
	// -> false

#### chunk(block, context) ####

根据block的返回结果将集合切分成组，每个组的元素都是相邻的，返回一个数组其元素中包含着当前block返回值和分成的组列元素。

	var list = new Collection([3, 1, 4, 1, 5, 9, 2, 6, 5, 3, 5]);
	list.chunk(function(value) { return value % 2 === 0 })
	
	// -> [
	//        [false, [3, 1]],
	//        [true,  [4]],
	//        [false, [1, 5, 9]],
	//        [true,  [2, 6]],
	//        [false, [5, 3, 5]]
	//    ]

#### collect(block, context) ####

其别名为 **map()**。

#### count(needle, context) ####

如果不带参数使用则会返回集合中的元素数量。如果带有参数时，使用 `equality` 语法来与参数 `needle` 进行比较返回和相等的元素个数，或者`needle` 为函数时只有符合返回真值时的元素个数。

	new Collection(3,7,4,8,2).count();
	// -> 5
	
	new Collection(3,7,4,8,2).count(2);
	// -> 1
	
	new Collection(3,7,4,8,2).count(function(x) { return x % 2 == 0 });
	// -> 3

#### cycle(n, block, context) ####

循环集合指定的 `n` 次，并且调用 `block` 执行每个成员。相当于，调用执行 `forEach(block, context)` 指定的 `n` 次。

#### detect(block, context) ####

其别名为 **find()** 。

#### drop(n) ####

返回一个新数组，但是只包含除去集合中的前 `n` 个成员后的所有。

#### dropWhile(block, context) ####

返回一个只包含block返回真的元素的新数组(去除掉block返回false的集合元素)。

#### entries() ####

别名为 **toArray()** 。

#### every(block, context) ####

别名为 **all()**

#### filter(block, context) ####

别名为 **select()**

#### find(block, context) ####

返回集合中首个符合block为true的元素。别名为 **detect()** 。

	new Collection(3,7,4,8,2).find(function(x) { return x > 5 });
	// -> 7

#### findAll(block, context) ####

别名为 **select()**

#### findIndex(needle, context) ####

返回等于 `needle` 的首个集合元素的索引值，或者`needle` 为函数时则返回为true的首个元素的索引值。若没有找到符合的则返回 `null` 。

#### first(n) ####

返回包含前 `n` 个元素的数组，或者没有指定参数`n` 时则只返回首个元素。

#### forEachCons(n, block, context) ####

按照每块指定的 `n` 个连续元素作为block的参数进行调用。

	new Collection(3,7,4,8,2).forEachCons(3, function(list) {
	    console.log(list);
	});
	
	// prints
	//    [3, 7, 4]
	//    [7, 4, 8]
	//    [4, 8, 2] 

#### forEachSlice(n, block, context) ####

与上面的 **forEachCons** 近似，不过是被指定每 `n` 个分隔一次然后依次进行调用block函数。

	new Collection(3,7,4,8,2).forEachSlice(2, function(list) {
	    console.log(list);
	});
	
	// prints
	//    [3, 7]
	//    [4, 8]
	//    [2]

#### forEachWithIndex(block, context) ####

遍历集合中的每一个元素，然后传递元素和其索引值到block中。

	new Collection(3,7,4,8,2).forEachWithIndex(function(x,i) {
	    console.log(x, i);
	});
	// prints
	//    3, 0
	//    7, 1
	//    4, 2
	//    8, 3
	//    2, 4

#### forEachWithObject(object, block, context) ####

遍历集合中每个元素，传递每个元素和指定的 `object` 到block中，最后返回当前的对象`object` 。

	var list = new Collection(3,7,4,8,2);
	
	list.forEachWithObject([], function(ary, item) {
	    ary.unshift(item * item);
	});
	// -> [4, 64, 16, 49, 9]

#### grep(pattern, block, context) ####

返回包含所有匹配 `pattern` 的集合元素的一个数组。 `pattern` 可能是一个正则对象，一个模块，类，范围对象，或其它任何带有 `match()` 方法的对象。如果提供了block，那么每个匹配的元素将作为参数传递给block.

	var strings = new Collection('iguana', 'labrador', 'albatross');
	
	strings.grep(/[aeiou]a/);
	// -> ["iguana"]
	
	strings.grep(/[aeiou]a/, function(s) { return s.toUpperCase() });
	// -> ["IGUANA"]

#### groupBy(block, context) ####

通过block函数返回值进行分组，然后返回一个 `hash` 对象，每队哈希值的键是block返回的值，而哈希值对的值则是一个包含产生键结果的元素的数组。

	var list   = new Collection(1,2,3,4,5,6);
	var groups = list.groupBy(function(x) { return x % 3 });
	
	groups.keys()   // -> [1, 2, 0]
	groups.get(1)   // -> [1, 4]
	groups.get(2)   // -> [2, 5]
	groups.get(0)   // -> [3, 6]

#### inject(memo, block, context) ####

通过一个回调函数将集合缩减一个单值。当block首次被调用执行时，首次传入的参数为指定的memo。然后block的返回值又成为下一次memo的值。

	// sum the values
	new Collection(3,7,4,8,2).inject(0, function(memo, x) { return memo + x });
	// -> 24

#### map(block, context) ####

返回一个由集合中每个元素经过block加工后生成的数组。别名为 **collect()**


	// square the numbers
	new Collection(3,7,4,8,2).map(function(x) { return x * x });
	// -> [9, 49, 16, 64, 4]

#### max(block, context) ####

返回集合中最大的元素值。每个元素使用 `Comparable` 或用JavaScript的标准比较运算符进行比较。如果提供了block则进行元素排序。若为提供block则按默认排序方式。

	var list = new Collection(3,7,4,8,2);
	
	list.max()   // -> 8
	
	list.max(function(a,b) { return (a%7) - (b%7) });
	// -> 4

#### maxBy(block, context) ####

通过调用block返回得到最大值的元素。

#### member(needle) ####

如果集合中任意元素等于指定的needle的话返回true。元素之间的检查通过全等 `===` 或在对象之间使用 `equals` 方法来实现。

	var list = new Collection(3,7,4,8,2);
	list.member('7')   // -> false
	list.member(7)     // -> true

#### min(block, context) ####

与 `max()` 类似，只是返回最小值。

#### minBy(block, context) ####

与 `maxBy()` 反之同理。

#### minmax(block, context) ####

返回数组 [min(block, context), max(block, context)]。

#### @minmaxBy(block, context) ####

返回数组[minBy(block, context), maxBy(block, context)]

#### none(block, context) ####

返回 !collection.any(block, context)。

#### one(block, context) ####

当block返回真的元素只有一个符合时返回true。如果为提供block的话，只含有一个真值元素时返回true。

#### partition(block, context) ####

返回两个数组，其中一个包含block返回真的所有元素，而另一个则包含其它block返回false的元素。

	new Collection(3,7,4,8,2).partition(function(x) { return x > 5 });
	// -> [ [7, 8], [3, 4, 2] ]

#### reverseForEach(block, context) ####

通过block函数遍历每个元素，以倒序传入 `forEach` 。

#### reject(block, context) ####

返回一个新的数组，其中包含的是 block 返回false的元素。

	new Collection(3,7,4,8,2).reject(function(x) { return x > 5 });
	// -> [3, 4, 2]

#### select(block, context) ####

与 `reject` 相反，其别为 `filter()` 和 `findAll()` 。

	new Collection(3,7,4,8,2).select(function(x) { return x > 5 });
	// -> [7, 8]

#### some(block, context) ####

别名为 `any()` 

#### sort(block, context) ####

返回新数组包含重新排序后的所有元素。每个元素使用 Comparable 或用JavaScript的标准比较运算符进行比较。如果提供了block则进行元素排序。若为提供block则按默认排序方式。

	var list = new Collection(3,7,4,8,2);
	
	list.sort()
	// -> [2, 3, 4, 7, 8]
	
	// sort by comparing values modulo 7
	list.sort(function(a,b) { return (a%7) - (b%7) });
	// -> [7, 8, 2, 3, 4]

#### sortBy(block, context) ####

返回新数组，其包括的所有元素根据block返回的结果进行排序。

	// sort values modulo 7
	new Collection(3,7,4,8,2).sortBy(function(x) { return x % 7 });
	// -> [7, 8, 2, 3, 4]

#### take(n) ####

返回集合中的前 `n` 个元素。

#### takeWhile(block, context) ####

从集合开头返回每个元素，直到第一个block返回false的元素为止且不包括该元素。

#### toArray() ####

返回包含所有集合元素的新数组。别名为 `entries()` 。

#### zip(args, block, context) ####

这个函数难以用语言描述的，可以借鉴Ruby的文档来进行解释一番：

将参数转换为数组，然后合并集合中相对应的元素。生成一系列的 `n` 多元素数组，这里的 `n` 指的是参数的数量。如果参数的数量小于集合元素数量时，则对应地返回 `null` 值。如果提供block的，将输出数组中的每个元素作为参数调用，否则直接返回数组。

	new Collection(3,7,4,8,2).zip([1,9,3,6,4], [6,3,3]);
	// -> [
	//        [3, 1, 6],
	//        [7, 9, 3],
	//        [4, 3, 3],
	//        [8, 6, null],
	//        [2, 4, null]
	//    ]
	
	new Collection(3,7,4,8,2).zip([1,9,3,6,4], function(list) {
	    console.log(list)
	});
	
	// prints...
	//    [3, 1]
	//    [7, 9]
	//    [4, 3]
	//    [8, 6]
	//    [2, 4]