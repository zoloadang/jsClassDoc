Package management 包管理器
===========================

简介
----

JS.Class自带了包管理器，它可以在所有支持JS.Class的平台上很容易地加载应用中所需要的类库。轻松指定请求你的代码中所需要的模块对象，及所需模块的依赖项，包管理器会帮你处理好所有的关系并成功加载。意即你的应用代码中只需要引入所需的模块对象就行，而不需要关心所需运行脚本下载问题。


比如，架设我需要Console模块，那么我只需要通过require方法调用它，一旦所有必需的代码成功请求到，立即运行提供的回调函数：

		JS.require('JS.Console', function() {
			JS.Console.puts('Hello, world!');
		});

包管理器只负责加载你所需要的对象，而且它会保证每个文件只会请求加载一次。在下载过程中支持并行加载以提升性能，而且同时保证依赖脚本按照正确顺序执行。

它被设计成可以支持从任意域名下加载代码，也包括那些自身拥有包管理系统的类库。虽然像 **CommonJS **框架在某些方向是一个很好的解决方案，但是它对于请求的源代码的编写方式有约定。如果使用  **JS.Packages **可以让你通过统一接口加载任何你需要的代码，无须对请求的源代码进行修改。毕竟在网络开发过程中频繁使用的代码需要保持稳定，不会随意开放编辑权限的。


 **基本使用案例： **
-----------------

_前提_

在使用  **require **语句之前，你需要在页面中加载如下内容：

 * JS.Packages -- 依赖管理器
 * 依赖列表

你可以通过加载  **package.js **或者  **loader.js **来正常使用JS.Packages。(loader.js是package.js与JS.Class自身依赖数据的集合；所以如果你只是想要使用JS.Packages进行管理代码并不需要使用JS.Class中其它的模块时，选择使用package.js)

当加载好包管理器后，需要设置代码路径了。

 ***

_ **包文件清单： **_

描述包配置，罗列出你的应用所需要的所有外部脚本文件，说明哪些JavaScript对象是由外部脚本  **provides **输出的，哪些对象是被依赖  **requires **请求的。例如，下面展示的是JS.Class库的一些模块：

		JS.Packages(function() { with(this) {
		    file(JSCLASS_PATH + '/core.js')
		        .provides('JS.Module',
		                  'JS.Class',
		                  'JS.Kernel');
		
		    file(JSCLASS_PATH + '/comparable.js')
		        .provides('JS.Comparable')
		        .requires('JS.Module');
		
		    file(JSCLASS_PATH + '/enumerable.js')
		        .provides('JS.Enumerable')
		        .requires('JS.Module',
		                  'JS.Class');
		
		    file(JSCLASS_PATH + '/hash.js')
		        .provides('JS.Hash',
		                  'JS.OrderedHash')
		        .requires('JS.Class',
		                  'JS.Enumerable',
		                  'JS.Comparable');
		}});

> __注意：__ 在上面列表中的  **Enumerable **依赖于  **Class **和  **Module **两个对象，经过观察我们可以发现，这两个模块是由同一个文件(core.js)提供输出的。同理，  **Hash **依赖于  **Enumerable **和  **Comparable **两个对象，而这两个对象又依赖于  **Module **对象。

> 像上面所叙述的这样错综的关系，包管理器都可以帮我们很好地进行处理，而且每个文件只会请求一次。常常通过对象检测来判断任意个文件是否已成功加载，当在检测过程中发现某个对象未定义则说明相应地文件没有被请求。

> 另外，包管理系统会尝试在多个脚本之间没有依赖关系情况下采取并行加载，因为它无需考虑执行顺序问题。例如，上面代码中的  **Enumerable **和  **Comparable **之间彼此没有关系，所以当我们想要加载  **Hash **时，这些依赖项就可以并行加载。如果在一系列脚本中加载顺序要慎重考虑时，必须要确信清楚地使用  **requires() **语句。只有当所有依赖项全部加载完毕才可以开始加载被依赖项。

> 每个包管理器也可以罗列多个文件。在这种情况下，由于JS.Packages不明白这些多个文件之间的关系，所以它不能自动判断使用并行加载，只能按照  **JS.require() **语句顺序依次加载。当处理多文件之间的相互关系令人头痛不已，而你如果觉得值得亲手去指定好这些文件加载顺序的话，那这个包可以提供一些方便。

例如，加载Fancybox库，你可以像下面这样做：

		JS.Packages(function() { with(this) {
		    file( 'fancybox/lib/jquery-1.7.1.min.js',
		          'fancybox/source/jquery.fancybox.pack.js',
		          'fancybox/source/helpers/jquery.fancybox-buttons.js',
		          'fancybox/source/helpers/jquery.fancybox-thumbs.js')
		        .provides('jQuery.fancybox');
		}});

> 除了使用  **requires() **外，还有另外个语句  **uses() **其用来指定一个"软依赖"，意思就是包需要一个对象但它不是非常必要的需首先加载的。例如，  **Set **需要使用  **Hash **来进行存储需求但是你只能在  **Set **包配置好后才能载入  **Hash **。换句话说，  **Set **需要混入  **Enumerable **而且需要先载入它。但是，  **Hash **它自身又基于  **Enumerable **。所以这个包配置看起来应该是这样子的：

		JS.Packages(function() { with(this) {
		    file(JSCLASS_PATH + '/enumerable.js')
		        .provides('JS.Enumerable');
		
		    file(JSCLASS_PATH + '/hash.js')
		        .provides('JS.Hash')
		        .requires('JS.Enumerable');
		
		    file(JSCLASS_PATH + '/set.js')
		        .provides('JS.Set')
		        .requires('JS.Enumerable')
		        .uses('JS.Hash');
		});


>   **提示： ** __uses()__的优势在于它可以帮助包系统优化下载，因为如果无所谓考虑载入顺序的话包可以采取并行下载，提升性能。

 **利用autoload快速设置 **
-----------------------

当你的应用逐步增长过程中可能会慢慢发现包配置中出现过多的重复。例如，在你的一批测试脚本中对应着一堆的类：

		JS.Packages(function() { with(this) {
		
		    file('tests/widget_spec.js')
		        .provides('WidgetSpec')
		        .requires('MyApp.Widget');
		
		    file('tests/blog_post_spec.js')
		        .provides('BlogPostSpec')
		        .requires('MyApp.BlogPost');
		
		    file('tests/users/profile_spec.js')
		        .provides('Users.ProfileSpec')
		        .requires('MyApp.Users.Profile');
		});

> 当你遇到类似于一些对象的名称可以通过模式来匹配的情况时，可以选择使用 **autoload() **函数来进行包配置。例如，针对上面的配置代码可以简化为：

		JS.Packages(function() { with(this) {
		    autoload(/^(.*)Spec$/, {from: 'tests', require: 'MyApp.$1'});
		});

>  **autoload() **接受三个参数。首位参数是一个正则表达式被用来匹配包名； **from **选项代表的是匹配的包名所在的目录位置，例如通过此种规则包加载器会在_tests/users/profile\_spec.js_中寻找 **Users.ProfileSpec **模块；另外的 **require **选项指定依赖项，通过正则来匹配。根据上面的规则举例说明则 **Users.ProfileSpec **的一个依赖为 **MyApp.Users.Profile **。

> 当 **require() **请求包且没有明确配置时，autoloader将会尝试通过匹配名来寻找加载。名称转换路径规范约定为：点符号(.)转换为路径分隔符(/)，驼峰式命名转换为下划线风格。例如，*Users.ProfileSpec* -> *users/profile_spec.js*

 **定制式的loader函数 **
---------------------

有些类库像[ **Google Ajax APIs **](http://code.google.com/apis/ajax/)，它们有属于自己的脚本文件搜寻加载系统。我们的包系统可以允许指定其它类库包，并通过 **loader **函数代替路径加载。 **loader **函数可以含带一个回调并且在当前类库加载完毕后调用。例如，下面是在你的库中结合Google Maps进行操作：

		JS.Packages(function() { with(this) {
		    file('http://www.google.com/jsapi?key=MY_GOOGLE_KEY')
		        .provides('google.load');
		
		    loader(function(cb) { google.load('maps', '2.x', {callback: cb}) })
		        .provides('GMap2', 'GClientGeocoder')
		        .requires('google.load');
		}});

> 回调函数(cb)是由包系统定义产生的，而且会在 **loader **函数完成固定任务后进行执行回调中的代码。如果不直接调用cb或者将其传入一个函数中以备调用，那么代码将不会执行。

 **JS.Packages **也提供了载后设置挂钩方法，意思是当文件加载好后执行一些代码。例如，加载[ **YUI3 **](http://developer.yahoo.com/yui/3/)时可能会包含种子文件，那么可以在包系统中生成一个全局YUI实例，然后再利用YUI自己的加载器装载其它更多的模块。如下所示：

		JS.Packages(function() { with(this) {
		    file('http://yui.yahooapis.com/3.0.0pr2/build/yui/yui-min.js')
		        .setup(function() { window.yui3 = YUI() })
		        .provides('YUI', 'yui3');
		
		    loader(function(cb) { yui3.use('node', cb) })
		        .provides('yui3.Node')
		        .requires('yui3');
		}});

 **Loader **函数也常常被用于在包中直接创建库对象，而不通过外部文件加载。只是要记住当生成对象准备好时调用cb回调函数：

		JS.Packages(function() { with(this) {
		    loader(function(cb) {
		        window.ChocolateFactory = new WonkaVenture();
		        // Perform other expensive setup operations
		        cb();
		    })
		    .provides('ChocolateFactory');
		}});


 **打包部署 **
-----------

在开发阶段通常的设置都是通过 **script **标签载入包管理加载器和包目录文件，然后再通过 **JS.require() **方法引入所要用到的组件。如下结构所示：

	<script type="text/javascript" src="/js.class/loader.js"></script>
	<script type="text/javascript" src="/manifest.js"></script>
	
	<script type="text/javascript">
	    JS.require('Application', function() {
	        // ...
	    });
	</script>

相应地，__manifest.js__文件中可能包含的内容如下所示：

	JS.Packages(function() { with(this) {
	    file('https://ajax.googleapis.com/ajax/libs/jquery/1.6.2/jquery.js')
	        .provides('jQuery');
	
	    file('/app.js')
	        .provides('Application')
	        .requires('jQuery');
	}});

在开发阶段可以像上面那样很简单地进行设置来加载所需要的模块，但是在生产环境下经常会把全部的JavaScript文件打包成一个文件载入页面。

_jsbuild_
----------

 **jsbuild **是基于Node环境而写的命令行程序。它将把你的包目录文件和所需要的模块列表作为输入，然后会输出一个包含那些模块及它们的依赖项的单独 **JavaScript **文件。它会在本地系统寻找所需要的模块，也可以通过网络载入外部的脚本等。

首先，要打包生成一个 **package **，通过__npm__安装 **JS.Class **:

	npm install --global jsclass

然后可以运行类似如下的命令：

	jsbuild --manifest MANIFEST --root ROOT [OPTIONS] module1 [module2 ...]

可用的参数如下：

  -  **--manifest, -m: ** 包目录清单(manifest.js)文件所在的路径。如果你只需要应用 **JS.Class **自身的模块，那么该参数为可选项。

  -  **--root, -r: ** 包含应用脚本文件的目录。清单(manifest)文件的路径都是相对于该目录解析的。

  -  **--external, -e: ** 如果该参数没有设置，那么在构建时会跳过需要外网提供的文件，不会影响本地文件系统。	

  -  **--no-packages, -P: ** 如果设置了该项，则 **JS.Packages **系统将不会被包含在构建文件中。包含的话意味着可以正常使用 **JS.require() **。

  -  **--bundles, -b: ** 可选择性地构建捆绑文件。

  -  **--output, -o: ** 输出类型,可以选择"code"(默认)或"paths"任一种。选择前者则会输出合并后的源代码，而选择后者则输出绑定文件路径。

  -  **--directory, -d: ** 配合  **-o paths **时使用，简短输出目录路径。比如，传入*-d public/js*时，使用 **jsbuild **命令后则会输出*app.js*而不用再输出*public/js/app.js*。

例如，构建一个包来支持我们的应用，我们可以运行下面所示的命令来生成一个包含 **JS.Packages,jQuery和Application **的脚本，因为 **Application **依赖于 **jQuery **:

	$ jsbuild --manifest public/js/manifest.js \
	          --root public/js/ \
	          --external \
	          Application

上面这段命令输入后生成的结果脚本会在标准输出窗口打印出来。请注意，使用 **jsbuild **过程中不会对文件进行任何压缩操作，它只是简单地搜寻所需要的模块文件并组合它们。如果需要进行压缩则单独相应处理。

如果只是希望构建包含 **JS.Class **部分模块的文件时，在运行 **jsbuild **时带上所需要的模块名即可：

	$ jsbuild JS.Set JS.Deferrable

组织构建文件
-----------

我们常常会遇到这种烦恼，随着项目地不断增长，会可能不需要所有的模块和依赖存在于一个单独文件中。我们可以尽量最小化HTTP请求的浪费，因为它意味着每次的修改都会造成构建文件的重新下载。基于这个原因，很多人采用的方式是将稳定的不变的类库资源与会频繁改动的应用代码独立分开。那么， **jsbuild **可以通过一个简单的*JSON*文件来实现上面所说的方式。

该*JSON*文件中罗列很多构建对象，每个对象都含有一个 **include **域和一个可选的 **exclude **域，这两个域可以一起使用也可以单独使用。 **include **域指定的是来自manifest的对象应该被构建的对象，而 **exclude **常用来指定跳过的独立加载依赖对象。

例如，看下面是如何分割稳定类库和应用代码的：

	// bundles.json
	{
	    "libs": {
	        "include": [ "jQuery" ]
	    },
	    "app": {
	        "exclude": "libs",
	        "include": "Application" 
	    }
	}

接下来，我们可以通过传递JSON文件来代替对象模块名来构建文件了。使用过程中要记得传递 **-b bundle.json **指定构建文件的具体位置。

	$ jsbuild -m public/js/manifest.js -b bundles.json -r public/js -o paths libs
	https://ajax.googleapis.com/ajax/libs/jquery/1.6.2/jquery.js
	
	$ jsbuild -m public/js/manifest.js -b bundles.json -r public/js -o paths app
	public/js/app.js

如果我们从 **app **构建中删除*"exclude":"libs"*这一行，jQuery文件会在在我们自己文件之前就被载入了。

	$ jsbuild -m public/js/manifest.js -b bundles.json -r public/js -o paths app
	https://ajax.googleapis.com/ajax/libs/jquery/1.6.2/jquery.js
	public/js/app.js