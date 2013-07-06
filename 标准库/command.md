# Command - 命令 #

### 概述 ###

**Command** 是命令模式的一种实现，其对象常用于描述行为。一个命令对象常含有一个 `execute()` 方法来运行其行为，而有时候需要 `undo()` 方法来进行撤销操作。

	// In the browser
	JS.require('JS.Command', function(Command) { ... });
	
	// In CommonJS
	var Command = require('jsclass/src/command').Command;

### 创建命令 ###

下面是最基础的 'Hello world'命令：

	var helloWorld = new Command({
	    execute: function() {
	        alert('Hello world!');
	    }
	});
	
	helloWorld.execute()
	// -> alerts "Hello world!"

如果想要在代码中通过 `undo` 方法来撤销行为，需要在代码中明确指定撤销动作：

	var incrementCounter = new Command({
	    execute: function() {
	        someCounter += 1;
	    },
	    undo: function() {
	        someCounter -= 1;
	    }
	});
	
	var someCounter = 0;
	incrementCounter.execute();
	incrementCounter.execute();
	incrementCounter.undo();
	
	// someCounter is now == 1

### 创建自定义命令类 ###

经常会想通过 **Command** 子类来提供自定义的命令。例如，想要创建一种命令，可以在任意 `canvas` 元素上画出一个圆圈：

	var DrawCircleCommand = new Class(Command, {
	    initialize: function(ctx) {
	
	        var x = Math.random() * 400,
	            y = Math.random() * 300,
	            r = Math.random() * 50;
	
	        this.callSuper({
	            execute: function() {
	                ctx.fillStyle = 'rgb(255,0,0)';
	                ctx.beginPath();
	                ctx.arc(x, y, r, 0, 2*Math.PI, true);
	                ctx.fill();
	            }
	        });
	    }
	});
	
	var ctx = myCanvas.getContext('2d');
	var randomCircle = new DrawCircleCommand(ctx);
	
	randomCircle.execute();

> 注意一下这个常见的模式：我们创建了一个 **Command** 的子类。该新子类会接受一个绘图上下文并进行初始化(ctx)，选择一些任意绘画起点，然后使用 `this.callSuper` 传递一些命令属性给命令初始化方法。另外需要明白的是任意数值只生成一次，并不是在每次命令被执行时就生成一次：每个 **new DrawCircleCommand** 会不同，但是每个单独的 **DrawCircleCommand** 实例应该在每次执行时做同样的实现。

**重要：** 在命令子类中不要尝试重写 `execute()` 或 	`undo()` 方法。因为它们和命令堆栈存在着联系而且规定不准被修改的。必须通过上面代码所示使用 `callSuper()` 来设置命令。不能按如下方式：

	var BrokenCommand = new Class(Command, {
	    execute: function() {
	        doSomething();
	    },
	    undo: function() {
	        undoSomething();
	    }
	});

> 这个命令如果挂钩到接下来要介绍的命令堆栈中就不能正常工作了。

### Command.Stack ###

如果在有很多行为情况下，支持回退撤销和存储命令历史，所以就可以延后回退和前进操作，这时可以使用 **Command.Stack** ，创建它并赋给其一个名称：

	var counterStack = new Command.Stack();

接下来你可以创建命令，然后无论何时被执行这些命令都会自动添加到堆栈中，使用 `stack` 选项即可：

	var incrementCounter = new Command({
	    execute: function() {
	        someCounter += 1;
	    },
	    undo: function() {
	        someCounter -= 1;
	    },
	    stack: counterStack
	});

现在，只要 **incrementCounter** 被执行，命令就会添加到堆栈 `counterStack` 中。通过使用 `undo()` 和 `redo()` 方法来控制回退和前进调用命令历史记录：

	var someCounter = 0;
	
	incrementCounter.execute();   // someCounter == 1
	incrementCounter.execute();   // someCounter == 2
	incrementCounter.execute();   // someCounter == 3
	
	counterStack.undo();          // someCounter == 2
	counterStack.redo();          // someCounter == 3
	counterStack.undo();          // someCounter == 2
	counterStack.undo();          // someCounter == 1

这种方式在有很多不同类型的命令时会更能发挥用处，而且也是一种逐步访问命令堆栈的手段方式。

命令堆栈还有其它一些方法我们应该了解掌握的：

**stepTo()** 可以根据提供的数字来回退到堆栈命令历史记录中任意点。

**stepTo(0)** 撤销整个堆栈，其中的命令逐一回退。

**push()** 允许自己推送命令到堆栈中 -- 这个是 `Command` 内部使用的方法，用于将命令与堆栈进行联系的。当推送一个命令到堆栈上时，该命令即会在正所在堆栈位置处进行添加，而且在添加处之后的任意还原命令会全部丢弃掉。例如：

	someCounter = 0;
	counterStack.clear();         // 0 commands in stack
	
	incrementCounter.execute();   // 1 command
	incrementCounter.execute();   // 2 commands
	incrementCounter.execute();   // 3 commands
	incrementCounter.execute();   // 4 commands
	                              // someCounter == 4
	
	counterStack.stepTo(2);       // still 4 commands in stack
	                              // stack pointer is after 2nd command
	                              // someCounter == 2
	
	incrementCounter.execute();   // stack truncated
	                              // contains 3 commands
	                              // pointer at end of stack
	                              // someCounter == 3

### 从头开始重做 ###

有些命令难以还原的，但是仍可以将其存储到堆栈中。一个例子就是上面提到的绘画圆圈命令：它是很难以‘重绘’一个圆圈的，因为在它出现在那之前无法判断该区域被什么填充着。在某些情况中，通过一个堆栈从初始位置开始直到指定位置进行重做也是很容易地实现'撤销'的操作的。当创建一个堆栈时传递了 `redo` 命令，则该堆栈将会认为你想要从头开始执行重做并且会自动调用 **redo** 命令，在进行重复执行它的历史之前清除干净所有的标记。

举一个例子，下面是一段程序让你在canvas元素上画出任意方块。

	// Canvas information
	var canvas = document.getElementById('canvas'),
	    ctx    = canvas.getContext('2d'),
	    W      = 400,
	    H      = 300;
	
	// Command for clearing the drawing area
	var clearCanvas = new Command({
	    execute: function() {
	        ctx.fillStyle = 'rgb(255,255,255)';
	        ctx.fillRect(0, 0, W, H);
	    }
	});
	
	// A redo-from-start stack
	var drawingStack = new Command.Stack({redo: clearCanvas});
	
	// Command for drawing a random square
	var DrawSquareCommand = new Class(Command, {
	    initialize: function(ctx) {
	
	        var x = Math.random() * W,
	            y = Math.random() * H,
	            r = Math.random() * 30;
	
	        this.callSuper({
	            execute: function() {
	                ctx.fillStyle = 'rgb(0,0,255)';
	                ctx.fillRect(x, y, r, r);
	            },
	            stack: drawingStack
	        });
	
	        this.name = 'Draw square at ' + x + ', ' + y;
	    }
	});

为了撤销一个步骤，绘画堆栈清除了canvas并且重新执行了所有的命令直到需要的那一步。注意到在绘画命令中包含的一行代码： `stack:drawingStack` 确保其被堆栈所记录了。想要任意绘制方块，可以按如下所示：

	new DrawSquareCommand(ctx).execute();

可以很容易地关联到一个接口按钮上，为其添加一个代码段来负责调用 `drawingStack.undo()` 方法。

像上面那样编码应用程序的优势是可以随意添加更多不同类型的绘画命令，而无需非常复杂高级的方法来撤销它们 - 你只需要将所有你的命令公布到一个单独的命令堆栈，然后使用堆栈来恢复修改。

注意到，在命令的 `initialize()` 方法内定义一个 **name** 属性 -- 其作用是可以允许我们在下一个例子中讲到的GUI中来描述相应命令的。

### 长度和指针(length and pointer) ###

所有的堆栈都含有一个 `length` 属性，通过该属性可以得知在任何时间点堆栈中包含的命令数量。当新命令被推送到堆栈中时该属性会相应进行改变 -- 但是 `undo()` 或 `redo()` 方法则不会影响堆栈长度。那么指针(pointer)的改变意味着什么呢 -- 数值代表着堆栈中正在执行的命令所处的位置。例如：

	var stack = new Command.Stack();
	var command = new Command({
	    execute: function() { ... }
	});
	
	// stack.length == 0
	// stack.pointer == 0
	
	stack.push(command);
	stack.push(command);
	stack.push(command);
	
	// stack.length == 3
	// stack.pointer == 3
	
	stack.undo();
	stack.undo();
	
	// stack.length == 3
	// stack.pointer == 1
	
	stack.push(command);
	
	// stack gets truncated
	// stack.length == 2
	// stack.pointer == 2

### 可观察（Observable）和可枚举(Enumerable)的 ###

**Command.Stack** 混入了 `Observable` 和 `Enumerable` 两大模块。这意味着你可以实现一个类似photoshop类型的行为历史记录，来观察堆栈：

	<ul id="drawHistory"></ul>
	
	<script type="text/javascript">
	    drawingStack.subscribe(function(stack) {
	        var list = document.getElementById('drawHistory'), str = '';
	        stack.forEach(function(command, i) {
	            var color = (i >= stack.pointer) ? '#999' : '#000';
	            str += '<li style="color: ' + color + ';">' + command.name + '</li>';
	        });
	        list.innerHTML = str;
	    });
	</script>

上面的例子创建了一个列表，当堆栈有所改变时会相应地进行列表更新。 `subscribe` 回调函数传入的参数是一个堆栈的引用，该堆栈在任何时候都可能执行了一个新的命令或撤销/重做一个命令。堆栈中含有一个 `pointer` 属性来指示堆栈中当前正处于活动的命令。所以可以很轻松地区分出每个命令是否已经撤销与否，而且为当前正处于活动期的命令赋予灰色调。`forEach` 回调函数用来遍历堆栈中每个命令及在堆栈中的索引值(从0开始计算)。


### 理解命令模式的简单例子 ###

> 下面的这段代码是针对JavaScript脚本实现的命令模式：

	/* The Invoker function */
	var Switch = function(){
	    var _commands = [];
	    this.storeAndExecute = function(command){
	        _commands.push(command);
	        command.execute();
	    }
	}
	 
	/* The Receiver function */
	var Light = function(){
	    this.turnOn = function(){ console.log ('turn on')};
	    this.turnOff = function(){ console.log ('turn off') };
	}
	 
	/* The Command for turning on the light - ConcreteCommand #1 */
	var FlipUpCommand = function(light){
	    this.execute = light.turnOn;
	}
	 
	/* The Command for turning off the light - ConcreteCommand #2 */
	var FlipDownCommand = function(light){
	    this.execute = light.turnOff;
	}
	 
	var light = new Light();
	var switchUp = new FlipUpCommand(light);
	var switchDown = new FlipDownCommand(light);
	var s = new Switch();
	 
	s.storeAndExecute(switchUp);
	s.storeAndExecute(switchDown);