测试工具 -- Testing tool
========================

概述
----

JS.Class自带了一个称作 **JS.Test**的测试框架。可以使用这个框架测试任意的JavaScript脚本，不局限于使用JS.Class编写的代码，而是被设计成可以在非常宽泛的平台轻松使用的。同时，当结合 `JS.Packages`使用时可以更容易在跨不同平台上进行组织和测试。在JS.Class支持的各个平台间无需进行适配完全兼容。

当在浏览器中运行 `JS.Test` 时，它可以自动通知Mozilla Lab的 `TestSwarm`项目，可以进行持续集成自动化测试配置。

另外，测试工具包含了 `JS.Console` 模块，因此可以使用 `puts()` 和其它方法将测试结果进行输出。

---

自带使用教程
===========

首先，这个教程先从如何建立项目，书写测试单元和在`Node.js`平台上运行开始介绍，之后会依次介绍在浏览器端和其它服务端平台的使用。

接下来，我们先创建一个项目目录结构，如下所示：

	project/
	    source/
	    test/
	        specs/
	        runner.js
	    vendor/
	        jsclass/
	            core.js
	            package.js
	            test.js
	            (etc)

`JS.Test`不需要很特殊复杂的项目结构布局，但是建议按照上面的目录结构来组织。 `source`目录中存在项目应用源代码脚本。 `test/specs`目录中存放对应着 `source`目录中文件的 `spec` -- 也就是我们将要写的测试代码。最后， `test/runner.js`这个是作为载入代码和启动测试的脚本文件，可以通过命令行调用或在浏览器中载入。

好的，让我们先给 `runner.js`文件添加一些代码，我们需要先加载JS.Class框架，然后设置测试文件的路径，最后运行测试代码。

	// test/runner.js
	
	JSCLASS_PATH = 'vendor/jsclass';
	require('../' + JSCLASS_PATH + '/loader');
	
	JS.Packages(function() { with(this) {
	    autoload(/.*Spec$/, {from: 'test/specs'});
	}});
	
	JS.require('JS.Test', function() {
	    JS.require('UserSpec', JS.Test.method('autorun'));
	});

> 注解： `autoload`语句指示 `JS.Packages`在 `test/specs`目录中查找可以匹配 `/.*Spec$/`模式的任意对象，例如 `test/specs/user_spec.js`文件中应该包含 `UserSpec`对象。最后载入 `JS.Test`，接着加载测试说明文件并通过 `JS.Test.autorun()`方法运行。

让我们先试着在Node环境下运行一下：

![](http://jsclass.jcoglan.com/images/test1.png)

> 注解：还没有创建测试说明文件，所以Node报错提示。

那么，我们创建一个空白的文件，_test/specs/user\_spec.js_，再重新运行：

![](http://jsclass.jcoglan.com/images/test2.png)

> 注解：Node又报错了，提示文件中没有任何代码。

所以，接下来让我们开始填充测试说明代码吧！

编写specs
---------

测试说明代码近似于流行的RSpec风格，使用嵌套的上下文环境。使用 `describe`方法来分断上下文环境，而使用 `it`方法创建测试语句块。在上下文环境中可以使用`before`和`after`钩子方法来设置和消除测试中需要的状态。在`before`语句块中将属性赋值为`this`则可以使其作为局部变量存在于测试中。

OK，让我们写一个简单的User类测试说明。

	// test/specs/user_spec.js
	
	JS.ENV.UserSpec = JS.Test.describe('User', function() { with(this) {
	    before(function() { with(this) {
	        this.user = new User('James');
	    }});
	
	    it('has a name', function() { with(this) {
	        assertEqual('James', user.getName());
	    }});
	}});

> 注解：我们需要将`UserSpec`作为全局变量，只有这样才可以通过`JS.Packages`访问到。在不同的平台中访问全局作用域会需要不同的代码，但是你可以使用`JS.ENV`来引用它便可跨多平台访问了。

如果我们再次运行一遍，则会得到一些有用的输出：

![meaningful output](http://jsclass.jcoglan.com/images/test3.png)

> 注解：我们得到两个错误提示，一个是来自`before`语句块中，由于`User`不存在；一个是来自测试语句，因为`user`变量未创建过。为了修复这些个错误，我们需要创建一个类对象，并且指示`JS.Packages`如何正确搜寻它。

在`source/user.js`文件中添加代码：

	// source/user.js
	JS.ENV.User = new JS.Class('User');

修改一下runner.js文件，说明`UserSpec`依赖于`User`，然后告诉runner.js到哪里寻找`User`类：

	// test/runner.js
	
	JSCLASS_PATH = 'vendor/jsclass';
	require('../' + JSCLASS_PATH + '/loader');
	
	JS.Packages(function() { with(this) {
	    autoload(/^(.*)Spec$/, {from: 'test/specs', require: '$1'});
	
	    file('source/user.js')
	        .provides('User')
	        .requires('JS.Class');
	}});
	
	JS.require('JS.Test', function() {
	    JS.require('UserSpec', JS.Test.method('autorun'));
	});

重新运行一遍：

![](http://jsclass.jcoglan.com/images/test4.png)

运行后发现只剩下一个错误：我们的User类中不存在测试中的_getName_方法。那么让我们完成我们的类然后通过测试吧。

再修改下source/user.js文件：

	// source/user.js
	
	JS.ENV.User = new JS.Class('User', {
	    initialize: function(name) {
	        this._name = name;
	    },
	
	    getName: function() {
	        return this._name;
	    }
	});

最后一次运行：

![](http://jsclass.jcoglan.com/images/test5.png)

我们终于通过了测试，好了，接下来我们就可以照猫画虎地进行扩展写测试语句了。在后面会看到提供了很多的断言列表。

在浏览器中运行测试
----------------

在浏览器中跑测试需要针对测试建立一个页面，并且同时我们需要分成两个代码段：一个是特定平台下的控制脚本加载代码，另一个是与平台无关的只负责跑测试的代码。需要从`test/runner.js`脚本中提取出特定平台下负责加载脚本的代码，另存在成`test/console.js`文件。添加一个常量`ROOT`来指定相对于测试页面的项目根路径。

应该分成如下所示的两个文件：

_1. test/console.js_

	JSCLASS_PATH = 'vendor/jsclass';
	require('../' + JSCLASS_PATH + '/loader');
	require('./runner');

_2. test/runner.js_

	JS.Packages(function() { with(this) {
	    var ROOT = JS.ENV.ROOT || '.'; 
	
	    autoload(/^(.*)Spec$/, {from: ROOT + '/test/specs', require: '$1'});
	
	    file(ROOT + '/source/user.js')
	        .provides('User')
	        .requires('JS.Class');
	}});
	
	JS.require('JS.Test', function() {
	    JS.require('UserSpec', JS.Test.method('autorun'));
	});


仍然需要通过终端窗口运行Node环境来执行`node test/console.js`命令。
好了，现在我们建立一个页面，在浏览器环境中实现类似于`test/console.js`的功能：

	<!-- test/browser.html -->
	
	<!doctype html>
	<html>
	    <head>
	        <meta http-equiv="Content-Type" content="text/html; charset=utf-8">
	        <title>Test runner</title>
	    </head>
	    <body>
	        <script type="text/javascript">ROOT = '..'</script>
	        <script type="text/javascript" src="../vendor/jsclass/loader.js"></script>
	        <script type="text/javascript" src="./runner.js"></script>
	    </body>
	</html>

保存在成网页`test/browser.html`，然后在任意浏览器中打开并查看运行结果。

![](http://jsclass.jcoglan.com/images/test6.png)

当在浏览器中开始运行时，`JS.Test`会自动通知`TestSwarm`启动持续集成测试工具。

在其它平台上进行测试
------------------

我们已经尝试在浏览器和Node环境中运行了测试，如果我们结合其它的工具跑测试会怎么样呢？比如 `Rhino`？

	~/project $ rhino test/console.js 
	js: uncaught JavaScript runtime exception:
	ReferenceError: "require" is not defined.

Rhino和其它类似于V8/SpiderMonkey等引擎均不支持`require()`方法加载文件，它们通过`load()`方法加载。所以，我们需要调整一下_test/console.js_中的代码：

	// test/console.js
	JSCLASS_PATH = 'vendor/jsclass';
	
	if (typeof require === 'function') {
	    require('../' + JSCLASS_PATH + '/loader');
	    require('./runner');
	} else {
	    load(JSCLASS_PATH + '/loader.js');
	    load('test/runner.js');
	}

修改后再运行的话，就可以在Rhino,V8,SpiderMonkey,Narwhal和Node环境下成功测试了。

但是，在一些平台框架中，比如RingoJS(基于Rhino平台的框架)中不使用`var`声明的变量不会转成全局的，而`JSCLASS_PATH`必须保证为全局变量才可以正常工作，可以通过`JS.ENV`来实现，但是在声明`JSCLASS_PATH`时尚未载入`JS.ENV`，那怎么解决呢？办法就是只能按照如下方式修改`JSCLASS_PATH`：

	// test/console.js
	(function() {
	    var $ = (typeof this.global === 'object') ? this.global : this;
	    $.JSCLASS_PATH = 'vendor/jsclass';
	})();

经过上面的调整，现在已经可以在大部分的平台上运行测试，但在Windows脚本宿主环境中不支持。若要针对Windows解决这个问题，需要先单独定义个`load()`函数，所以最终适配的代码如下所示：

	// test/console.js
	
	//================================================================
	// Set up load() function for Windows Script Host
	
	if (this.ActiveXObject)
	    load = function(path) {
	        var fso = new ActiveXObject('Scripting.FileSystemObject'),
	            file, runner;
	
	        try {
	            file   = fso.OpenTextFile(path);
	            runner = function() { eval(file.ReadAll()) };
	            runner();
	        } finally {
	            try { if (file) file.Close() } catch (e) {}
	        }
	    };
	
	//================================================================
	// Set up JSCLASS_PATH variable
	
	(function() {
	    var $ = (typeof this.global === 'object') ? this.global : this;
	    $.JSCLASS_PATH = 'vendor/jsclass';
	})();
	
	//================================================================
	// Load the JS.Class package manager and test runner
	
	if (typeof require === 'function') {
	    require('../' + JSCLASS_PATH + '/loader');
	    require('./runner');
	} else {
	    load(JSCLASS_PATH + '/loader.js');
	    load('test/runner.js');
	}

> Well done!! 现在可以在所有支持平台上运行测试了！

----

断言目录
-------

在一个测试语句中，可以使用接下来介绍的断言函数列表中任意一个来验证结果。在运行过程中，如果断言失败则会立即提醒报错。

 1. `assertBlock([message, ]callback)`
 > 如果回调函数返回真值则通过验证。在所有的断言中_message_作为可选参数，它的用处是当在断言失败时可以自定义输出提示信息。

 2. `assert(boolean[, message])`
 > 当表达式_boolean_为真值时验证通过。

 3. `assertEqual(expected, actual[, message])`
 > 验证是否相等。期望值如果类属于`Equality`则会调用`equals()`方法，否则直接使用全等操作符。这个断言可以进行深层次的比较，比如数组和对象等。

 4. `assertNotEqual(expected, actual[, message])`
 > 与上同理反之

 5. `assertSame(expected, actual[, message])`
 > 当期望值与实际值为相同对象时验证通过。比如，@expected === actual@。

 6. `assertInDelta(expected, actual, delta[, message])`
 > 当|expected - actual| <= delta时验证通过，注意所有值均为数字类型。

 7. `assertKindOf(type, object[, message])`
 > 当object为type的实例对象时通过验证。type可以用js类型字符格式，类似'boolean'或'object'等，要么是一个类或模块的引用。

 8. `assertMatch(pattern, string[, message])`
 > 当string匹配正则表达式pattern时通过验证。同时也经常用于对象匹配，比如assertMatch(new JS.Range(1,10), 5).

 9. `assertNull(object[, message])`
 > 当对象为null时通过验证。相反的断言，assertNotNull

 10. `assertRespondTo(object, method[, message])`
 > 当object含有一个属性或方法名为method时成立。method使用字符表示

 11. `assertThrows(error[, error2], callback)`
 > 当回调函数执行抛出的错误符合给定任意一个时成立，比如assertThrows(TypeError, function() { null.foo() })

 12. `assertNothingThrown([error, ][message, ]callback)`
 > 当回调函数执行时不抛出错误时成立，比如assertNothingThrown(function() { 1 + 1 }).也可以指定抛出的错误类型，例如assertNothingThrown(TypeError, ReferenceError, function() { 1 + 1 });如果回调函数抛出的异常不在指定错误范围内时则直接提示错误error，而不是失败failed.

----

上下文语句块
-----------

JS.Test测试文档由嵌套上下文语句组成，类似于RSpec，Jasmine和其它测试框架。上下文由`describe`语句(也可以用`context`代替)组成。在每个语句块中可以含有多个`before`和`after`语句，代表着描述之前和之后的操作。通过使用`it`或`should`来添加测试语句。

每段测试会按照如下顺序执行：

 * `before`语句在上下文环境中由外及内地执行。
 * 测试语句(it or should)自动运行。
 * `after`语句在上下文环境中依序由内及外执行。

_Example_ : 下面的测试输出为：

	ThingSpec = JS.Test.describe("thing", function() { with(this) {
	    before(function() { this.puts("outer before") });
	    after (function() { this.puts("outer after") });
	
	    it("has tests", function() { with(this) {
	        puts("outer test");
	    }});
	
	    describe("nested", function() { with(this) {
	        before(function() { this.puts("INNER before") });
	        after (function() { this.puts("INNER after") });
	
	        it("does something", function() { with(this) {
	            puts("INNER test");
	        }});
	    }});
	}});


_Output:_

	outer before
	outer test
	outer after
	
	outer before
	INNER before
	INNER test
	INNER after
	outer after

 > 注解：在上面测试代码中的`before`和`after`语句块根据测试语句`it`，每一个it语句运行则before、after也各执行一次。针对_before_语句常用于在测试前创建或修改对象，而_after_语句则用来复原测试过程中的修改，扫尾工作等，例如：清除或关闭数据库。

----

`Mocking` 和 `stubbing`
----------------------------

Mocking和stubbing在面向对象程序的测试中有着很重要的作用。`JS.Test`帮助你在写测试用例时可以很简单方便地使用这两个技术。这两个术语理解起来还是有些困难的，所以今天根据我自己的理解来解释一下。

`Stubbing` -- 意思就是使用模拟的硬编码响应来代替一个方法，函数或者一个带有版本的完整对象。典型的应用就是用来隔离各个组件之间的关系及依赖的外部代码。例如，stubbing常用于将测试与数据库解耦，而使用硬编码来模拟请求结果进行代码测试。

`Mocking` -- 是一种行为测试，通过检查被调用执行的方法来核对相应的行为。类似于stubbing，它也是通过伪造一个方法来代替真实的方法，但它不同于stubbing的是在被调用执行的方法中可以设置期望值。所以，它常用于应用层和临界值之间的测试。

 > 在测试过程中可以在任意地方使用mocks和stubs，同时`JS.Test`会在每个测试结尾处删除stub方法，恢复成原始真实的方法。

------

`Stubbing方法使用`

我们首先介绍stubbing，因为它与mocking共享了很多的API，同时它也稍微简单一些。那么，我们先通过`stub()`函数来给一个对象stub个方法，如下所示：

	stub(object, 'methodName');
	object.methodName() // -> undefined

> 注解：上面是一个最简单的stub，这段代码的意思是调用`object.methodName()`带有任意参数时皆返回`undefined`而且无附属问题。但是，你可以应用`returns`修饰符来给它指定一个返回值。如果当你给其指定了多个返回值时，将会依次被调用，若达到返回值列表最后时会进行循环到开头。

	stub(object, 'methodName').returns('hello');
	object.methodName() // -> 'hello'
	
	stub(object, 'methodName').returns('many', 'return', 'values');
	object.methodName() // -> 'many'
	object.methodName() // -> 'return'
	object.methodName() // -> 'values'
	object.methodName() // -> 'many'

在很多的JavaScript方法中接受回调函数来代替直接返回值；同样你可以使用`yields`修饰符来stub。`yields`携带的参数列表传递给回调函数作为参数使用。同`returns`一样，你照样可以指定多个参数列表以进行循环。

	stub(object, 'methodName').yields(['some', 'args']);
	
	object.methodName(function(a, b) {
	    // a == 'some'
	    // b == 'args
	});
	
	stub(object, 'methodName').yields(['some', 'args'],
	                                  ['more', 'data']);
	
	object.methodName(function(a, b) {
	    // a == 'some'
	    // b == 'args'
	});
	
	object.methodName(function(a, b) {
	    // a == 'more'
	    // b == 'data'
	});

方法通过`yields`修饰符将回调函数的参数进行同步，而且通常期望一个回调函数作为最后的参数或倒数第二位的参数。也可以在调用函数后面指定一个上下文环境对象来将`this`和回调函数绑定到一起。如果方法被调用时回调函数没有在正确的环境中执行，则会抛出错误。

最后，在调用过程中你可以抛出自定义错误；使用`raises`修饰符便可实现：

	stub(object, 'methodName').raises(new TypeError());

------

`输入参数的作用`

通常我们想要通过不同的参数得到多种多样的输出结果，或者验证一个方法是否正确地调用了相应的参数。你可以在`returns,yields或raises`修饰符之前使用`given`修饰符来达到目标。例如，我们可以根据不同的输入返回不同结果：

	stub(object, 'methodName').given(2,2).returns(4);
	stub(object, 'methodName').given(1,2,3).returns(6);
	
	object.methodName(2,2)    // -> 4
	object.methodName(1,2,3)  // -> 6
	
	object.methodName()       // -> error

现在，我们的存根方法根据输入的参数得到指定的结果。如果在调用时的参数不是指定的，那么就会抛出错误提示。如果你希望在任意输入参数时皆返回*undefined*时，那么在测试设置中不要是使用修饰符`stub(object,'methodName')`。

当使用`yields`时，`given`被用于回调函数之前的调用参数。例如，我们可以stub一下JQuery的异步请求接口：

	stub(jQuery, 'get').given('/foo.html').yields(['foo']);
	stub(jQuery, 'get').given('/bar.html').yields(['bar']);
	
	jQuery.get('/foo.html', function(response) {
	    // response == 'foo'
	});
	
	jQuery.get('/bar.html', function(response) {
	    // response == 'bar'
	});

----

`参数匹配器`

有时候不能预先知道具体的参数值，或者只是关心参数的一小部分属性，像参数类型或DOM元素数组等。基于以上原因，当在stubbing时匹配符合输入数据时提供了一批匹配器以供使用。例如，我们可以设置一个stub，其代表一个包含字符"test"的数组后面带有任意多个参数：

	stub(object, 'methodName').given(arrayIncluding('test'), anyArgs()).returns(true);
	
	object.methodName(['foo', 'test', 'bar'], 'something')  // -> true
	object.methodName(['foo', 'bar'], 'something')          // -> error

这一整套的匹配器如下所述：

 * `anything()` 匹配任意单值
 * `anyArgs()` 匹配参数列表后部的任意数量的值(包括空值)
 * `instanceOf(type)` 匹配给定类型的值，例如：instanceOf('string') 或 instanceOf(JS.SortedSet)
 * `arrayIncluding(value[, value2, ...])` 匹配一个包含所有给定值的数组
 * `objectIncluding({key: value[, value2, ...]})` 匹配一个包含所有给定键值对的对象。
 * `match(pattern)` 匹配任意符合pattern的值。`pattern`可以是`RegExp`对象或者任意含有match()方法的对象，比如一个模块或一个范围类等。

-----

`Stubbing全局对象`

常常需要stubbing一个全局函数或对象来进行单元测试。那么，对于stubbing全局时，就是在`stub()`函数中省略对象参数。

	// Stubs out alert()
	stub('alert').given(instanceOf('string')).returns(undefined);

当stub一个全局对象时，就可以在你想要打断的位置给stub函数传入一个伪造的对象。例如，假设你有一些依赖于JQuery的Ajax接口的代码，你可以创建一个伪造的对象，然后再添加stub函数。记住，`JS.Test`在每个测试之后会清除创建的stubs。

	stub('jQuery', {});
	stub(jQuery, 'get').given('/foo.html').yields(['foo']);

----

`Stubbing构造函数`

构造函数也是一种函数，通过关键字`new`来构建对象。`JS.Test`支持Stub构造函数，其方式是`new`关键字作为stub函数的首位参数，然后依次是该构造函数的命名空间及名称。例如，下面介绍的是如果stub一个`JS.Range`返回模拟对象的：

	stub('new', JS, 'Range').returns({fake: 'object'})

如果当构造函数为全局变量时，那么stub函数中可以省略命名空间，例如：

	stub('new', 'XMLHttpRequest').returns(fakeXHR)

----

`Mocking方法`

Mocking非常类似于stubbing,事实上mocking等于stub加上验证其是否被调用。为即将被调用的方法设置一个mock期望，我们使用`expect()`函数代替`stub()`。

	expect(object, 'methodName');

上面这个mock状态为`object.methodName()`至少被调用一次；如果没有被调用执行过就说明该测试失败返回。

当创建mock时可以使用上面介绍的所有stub函数接口，但是不同的地方是在应用`expect()`函数时，如果设置的stubs没有完成调用，那么`JS.Test`会进行提醒。例如，在测试过程中调用JQuery.get('/foo.html', function(){...})首先先完成请求，然后再调用回调函数并响应结果为‘Hello, World’：

	expect(jQuery, 'get').given('/foo.html').yielding(['Hello, World']);

 > returns,yields和raises在设置mock期望时用returning,yielding和raising来代替，因为这样可读性更好一些。

可以通过修饰符`atLeast,atMost,exactly`来指定调用的次数。但是要注意，这些修饰符应该放在`given`修饰符后使用。例如，测试要求调用2次`User.create('jcoglan')`并每次都返回`true`:

	expect(User, 'create').given('jcoglan').exactly(2).returning(true);

----

`Stubbing`时间设定
-------------------

在编写JavaScript代码过程中在不同平台下会大量地使用到全局时间函数 - `setTimeout()`等。当遇到测试这种带有时间函数的代码时，通常会利用控制时钟来驱动测试正常前进；也就是说，在测试中设置了等待时间则需要完成时间再执行事件，那么可以自己控制调整时钟速度完成等待时间。这样就极大提高了测试速度，并且可以减少在进行异步测试过程中的干扰。

`JS.Test.FakeClock` 可以帮助你打断JS中的时钟函数，并且可以自由控制时间进度。但使用这个方法时，需要在测试代码中添加一些必要的钩子：

	JS.ENV.TimerSpec = JS.Test.describe('Timers', function() { with(this) {
	    include(JS.Test.FakeClock);
	
	    before(function() { this.clock.stub() });
	    after(function() { this.clock.reset() });
	
	    // Your tests here
	}});

> 调用`clock.stub()`方法代替了全部的全局时间函数 -- 如Date(),setTimeout(),clearTimeout(),setInterval()和clearInterval()等。所有这些全局时间函数都被模拟的时钟所控制，工作方式没有任何变化，仅仅是时间控制由计算机主导变成了自己。

当仅仅应用`FakeClock`时，定时器是不会运转的，还需要进行时钟设定才可以开启定时器。开启的方法是使用`clock.tick()`：

	// Advances time 1000 milliseconds
	this.clock.tick(1000);

好了，下面让我们看一个简易的例子。通过使用FakeClock测试在一定时间后进行改变字符串值的动作：

	JS.ENV.TimerSpec = JS.Test.describe('Timers', function() { with(this) {
	    include(JS.Test.FakeClock);
	
	    before(function() { this.clock.stub() });
	    after(function() { this.clock.reset() });
	
	    before(function() { with(this) {
	        this.string = 'foo';
	        JS.ENV.setTimeout(function() { string = 'bar' }, 100)
	    }});
	
	    it('changes a string', function() { with(this) {
	        assertEqual('foo', string);
	        clock.tick(100);
	        assertEqual('bar', string);
	    }});
	}});

> 注意：在上面代码段中我们看到，`setTimeout()`存在于`JS.ENV`命名空间，引用到一个全局对象上 -- 换言之，即浏览器中的`window`对象。之所以将变量`setTimeout`绑定起来，是防止在IE浏览器中修改`window.setTimeout`时被覆盖，所以解决方案就是要明确绑定到全局对象(另一个命名空间)上。如果不考虑在IE中使用`FakeClock`的话，那么完全可以省略`JS.ENV`引用了。
