# Loading your scripts #

### 概述 ###

`jstest` 不关心是如何加载你的代码的，因此可以随意使用任何的模块加载器。但是，如果你不想使用模块加载器，而想要在客户端和服务器端同时运行测试，那么这时 `jstest` 就可以发挥作用了。

在[入门指南](./getting_started.md)中，介绍了在页面中使用 `script` 标签载入脚本的方式。在一些平台中，这些脚本可能会被缓存起来，造成测试的不准确性。另外，如果你想要在服务器端运行测试，或者使用其它浏览器测试平台时，你需要在 `script` 标签中复制一些脚本加载的API信息。

`jstest` 通过使用一个程序式的API帮助解决上面提到的两种问题。创建 `example/runner.js`：

	// example/runner.js
	
	var run = function() { JS.Test.autorun() }
	
	var ROOT = JS.ENV.ROOT || '.'
	JS.cache = false
	
	JS.load(  ROOT + '/example/lib/set.js',
	          ROOT + '/example/spec/set_spec.js',
	
	          // add files here as the project grows
	
	          run)

> **JS.ENV** 引用的是全局对象，无论是在哪种平台上运行 `jstest` 都可以使用它轻松访问到全局变量；

通过上面提到的方式来重写[入门指南](./getting_started.md)中的页面，代码将会被自动载入，而且忽略浏览器缓存：

	<!-- example/browser.html -->
	
	<!doctype html>
	<html>
	  <head>
	    <meta http-equiv="Content-Type" content="text/html; charset=utf-8">
	    <title>jstest</title>
	  </head>
	  <body>
	
	    <script src="../build/jstest.js" type="text/javascript"></script>
	
	    <script type="text/javascript">ROOT = '..'</script>
	    <script src="./runner.js" type="text/javascript"></script>
	
	  </body>
	</html>

可以在[服务器端](./server.md)重用这个脚本加载代码，而且同样适用于各种各样的[浏览器测试平台](./browser.md)，能够帮助你将测试配置信息最小化。