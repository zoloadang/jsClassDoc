# Console - 控制台 #

### 概述 ###

大部分的JavaScript环境都有控制台工具，在执行代码时可以打印输出，日志信息等等。在Web浏览器中，通过提供 **console** 对象，而在服务器端环境中直接在标准终端上进行输出打印。`Console` 模块提供了一个单独接口来实现所有这些功能，适用于大部分的设备上。

	// In the browser
	JS.require('JS.Console', function(Console) { ... });
	
	// In CommonJS
	var Console = require('jsclass/src/console').Console;

当打印输出到标准输出终端或Webkit开发控制台时， **Console** 模块提供很多中格式命令，来改变颜色和打印出的文字的其它属性。

下面介绍下主要的两个输出方法：

 * **Console.puts(string)** -- 支持多行打印字符串到控制台，每行以换行符结束。
 * **Console.print(string)** -- 同上反之，单行输出；如果控制台支持该方法时可以同时频繁调用 `print()` 方法打印输出到同一行，反之则输出缓存直到下一个 `puts()` 调用。

输出设备按照如下方式使用：

 1. 在web浏览器中支持 `window.console`，所以开发控制台常被用于日志输出。
 2. 同第一点若浏览器没有 `console` 对象时，采用 `alert()` 方法代替。
 3. 在 `adl` 下运行Aodbe AIR应用时，通过使用 `runtime.trace()` 函数来输出。
 4. 运行在终端中的程序中，输出会借助于 `stdout`。


### 格式化 ###

下面的格式化命令用来格式化输出。不是所有的环境都支持格式化的，在那些不支持的环境中下面提到的这些命令会被忽略掉。所有的命令都是 **Console** 的方法，例如可以通过 `Console.bold()` 方法来切换到粗体文本。

 - `reset()` -- 重置所有个格式属性
 - `bold(), normal()` -- 设置文本的weight
 - `italic(), noitalic()` -- 在斜体和罗马字体之间切换
 - `blink(), noblink()` -- 应用或删除闪光文本
 - `underline(), noline()` -- 应用或删除下划线
 - `black(), red(), green(), yellow(), blue(), magenta(), cyan(), white(), nocolor()` -- 设置当前文本颜色
 - 颜色设置方法可以带有 `bg` 前缀，例如 `bgyellow()`，用来设置当前背景色


多种格式结构可以组合使用：

	Console.consoleFormat('bold', 'red');

这个 **consoleFormat()** 方法带有一系列格式结构作为输出。它首先会调用 `reset()` 方法，然后再应用所指定的格式。

在一些平台中， **Console** 支持另一些命令，像移动鼠标和清除屏幕等，在终端中也创建了接口。那些命令是：

- `cursorUp(n),cursorDown(n),cursorForward(n),cursorBack(n)` -- 在指定方向上移动鼠标到n处
- `cursorNextLine(n),cursorPrevLine(n)` -- 移动鼠标到指定n行
- `cursorColumn(x)` -- 移动鼠标到指定n列
- `cursorPosition(x,y)` -- 移动鼠标到指定的x列,y行
- `cursorHide(), cursorShow()` -- 隐藏和显示鼠标
- `eraseScreen()` -- 清除整个终端屏幕
- `eraseScreenForward(), eraseScreenBack()` -- 清除鼠标位置之前或之后所有
- `eraseLine()` -- 清除当前行
- `eraseLineForward(), eraseLineBack()` -- 清除当前行的鼠标左侧或右侧所有内容


### Console作为一个混入模块 ###

最后注意一下，**Console** 也可以作为混入模块将其所有方法应用到当前类中：

	Logger = new Class({
	    include: Console,
	
	    info: function(message) {
	        this.bgblack();
	        this.white();
	        this.print('INFO');
	        this.reset();
	        this.puts(' ' + message);
	    }
	});