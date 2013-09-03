# Browser testing #

### 概述 ###

在浏览器中运行测试是很简单的，只需要在页面加载代码和测试，接着直接调用 `JS.Test.autorun()` 就可以搞定了。在前面的[入门指南](./getting_started.md)和[Loading your code](./load_scripts.md) 两篇教程中都已经介绍过了。

一旦你已经了解这些知识点后，我们就可以介绍一些非常流行的测试框架如何来结合到你的测试代码中，进行运行检测了。当我们准备开始接触这些工具时，我们先测试一些简单的静态HTML页面，加载一些静态脚本文件。如果你的页面根本不需要额外的工具辅助的话，那么就更容易在其它别的方式中进行测试，例如使用 `PhantomJS` 进行 **CI** (持续集成)处理。

### HTML fixtures ###

如果想要能够保持可灵活性地运行测试，应该尽可能少的在页面和代码运行环境中假设处理。不能假设在主页面中可以任意修改初始化的HTML代码。同时也不要假设可以从服务器端加载代码。各式各样的工具不会让你同时这样做的。

因为这些原因，建议在测试框架中保持HTML固定结构。例如这里有一些小组件的交互的例子：

	JS.Test.describe('Widget', function() { with(this) {
	  fixture(' <div class="test-widget">\
	              <p>Hello world!</p>\
	              <ul></ul>\
	            </div>' )
	
	  before(function() { with(this) {
	    new Widget('.test-widget')
	  }})
	
	  it('adds a list item when the text is clicked', function() { with(this) {
	    $('.test-widget p').click()
	    assertEqual( 'added', $('.test-widget ul li').html() )
	  }})
	}})

**fixture()** 是一个辅助方法，其作用是当创建一个 **before()** 钩子函数时将HTML动态的添加到页面中，这些HTML被一个 `div` 标签包裹，最后在测试运行结束后将会删除所有的HTML。

	// spec/helpers.js
	JS.Test.Unit.TestCase.extend({
	  fixture: function(html) {
	    this.before(function() {
	      var holder = $('#fixture')
	      if (holder.length === 0) {
	        holder = $('<div id="fixture"></div>')
	        $('body').append(holder)
	      }
	      holder.html(html)
	    })
	
	    this.after(function() {
	      $('#fixture').empty()
	    })
	  }
	})

> 至于如何创建你的HTML结构取决于你自己， `jstest` 不会假设指定你在应用设置时使用的是JQuery还是其它什么框架。上面介绍的辅助函数在这里只是一个例子而已。

### Stubbing the server ###

假设，当我们点击组件插件时不使用硬编码它的HTML，而是请求服务器呢？就像下面这样：

	$('p').click(function() {
	  $.get('/message', function(response) {
	    $('ul').append('<li>' + response + '</li>')
	  })
	})

对上面这种情况进行单元测试的话，我们需要“打断”服务器交互，然后可以使用jstest的[stubbing API](../test.md)来实现。我们只需要在 `before()` 块中添加一行代码即可：

	before(function() { with(this) {
	    new Widget('.test-widget')
	    stub($, 'get').given('/message').yields(['added'])
	  }})

现在通过在合适位置进行“打断”可以让测试顺利执行通过。但是如果我们的代码使用 `promise-style` 异步请求呢？？

	$('p').click(function() {
	  $.get('/message').then(function(response) {
	    $('ul').append('<li>' + response + '</li>')
	  })
	})

这种情况也很容易利用“打断”的，我们可以创建一个预先解决的“承诺”模式包含着响应返回的结果，然后使用JQuery返回。

	before(function() { with(this) {
	    new Widget('.test-widget')
	
	    var promise = new $.Deferred()
	    promise.resolve('added')
	    stub($, 'get').given('/message').returns(promise)
	  }})