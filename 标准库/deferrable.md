# Deferrable - 延迟 #

### 概述 ###

**Deferrable** 模块被用于描述 `futures` 或 `promises` ，结果暂时是未知的。例如，**Ajax** 异步请求可以看作是一种延迟，因为直到响应被创建之前是未知的，而且回调函数只有当响应到达时才可以开始使用。**Deferrable** 模块在对象上如何添加和触发回调函数提供了API。

	// In the browser
	JS.require('JS.Deferrable', function(Deferrable) { ... });
	
	// In CommonJS
	var Deferrable = require('jsclass/src/deferrable').Deferrable;

### 建立一个延迟对象 ###

延迟对象的职责是监视一个需要长时间计算，当计算完成时提供一种方式通知进行相应的操作。让我们以Ajax请求为例吧：**Deferrable** 提供的API在客户端注册一个回调函数，然后当完成请求后我们的代码调用 `this.succeed()` 方法操作请求的结果。

	var AjaxRequest = new Class({
	    include: Deferrable,
	
	    initialize: function(url) {
	        var self = this;
	        jQuery.get(url, function(response) {
	            self.succeed(response);
	        });
	    }
	});

客户端通过这个类初始化一个URL然后添加回调函数。当这个类调用了 `succeed()` 方法后客户端回调函数才会被执行。

	var request = new AjaxRequest('/index.html');
	request.callback(function(response) {
	    // handle response
	});

每个被添加到延迟对象中的回调函数在调用下一个 `succeed()` 方法之前只会执行一次。如果当添加一个回调函数时，正好延迟已经完成，那么该回调函数会和最近被调用的 `succeed()` 返回值立刻被执行。

同时，**Deferrable** 模块还提供了一个基于回调的错误操作方法。想要通知一个错误，就可以使用延迟对象的 `errback()` 方法来添加回调。当延迟对象的 `fail()` 方法被调用了注册的报错回调就会被执行。

下面会对 **Deferrable** 模块提供的所有API罗列介绍。在要介绍的这些方法中， `block` 指的是一个函数，而 `context` 是一个可选项参数，指定了在 `block` 被执行时的上下文环境，即 **this** 的绑定。

#### callback(block, context) ####

为对象添加回调。如果对象已经接收到一个 `succeed()` ，那么回调函数会根据最后调用 `succeed()` 返回的值一起立即被执行，而不会添加到对象中。

#### errback(block, context) ####

为对象添加一个错误回调。与 `callback` 同理，只不过是当接收到 `fail()` 时立即执行。

#### timeout(milliseconds) ####

为对象设定一个时间限制(以毫秒为单位)。如果在设定的限制时间段中对象没有接收到 `succeed()` 或者 `fail()` 的话，那么 `fail()` 和 **Deferrable.Timeout** 错误将会被调用。

#### cancelTimeout() ####

消除延迟对象的时间限制。

#### succeed(value[, value2 ...]) ####

将对象状态设置为成功状态，然后执行任意的通过参数传递进来的回调函数。执行完成后会解绑回调函数，将不会再执行。

#### fail(value[, value2 ...]) ####

同 `succeed`反理！