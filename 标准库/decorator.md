# Decorator - 装饰者 #

### 概述 ###

**Decorator** 模块为我们提供了一套使用最少模版和代码重复来实现装饰者模式的方法。当创建一个装饰者类时，只需要定义一些不同于被装饰者(公共组件)对象中的方法即可。也就是说不需要再动手书写大量的转发方法，这样就可以节省大量时间，减轻文件大小和降低代码重复率。

	// In the browser
	JS.require('JS.Decorator', function(Decorator) { ... });
	
	// In CommonJS
	var Decorator = require('jsclass/src/decorator').Decorator;

接下来，让我们快速看一个实例解释：

	// Basic Bike class. Bikes cost $10 per gear.
	
	var Bike = new Class({
	    initialize: function(model, gears) {
	        this.model = model;
	        this.gears = gears;
	    },
	    getModel: function() {
	        return this.model;
	    },
	    getPrice: function() {
	        return 10 * this.gears;
	    },
	    applyBrakes: function(force) {
	        // slow the bike down...
	    }
	});
	
	// Disk brake decorator. Disk brakes add to the price, and make the bike's
	// brakes more powerful.
	
	var DiskBrakeDecorator = new Decorator(Bike, {
	    getPrice: function() {
	        return this.component.getPrice() + 50;
	    },
	    applyBrakes: function(force) {
	        this.component.applyBrakes(8 * force);
	    }
	});

`DiskBrakeDecorator` 获取到所有的 `Bike` 的实例方法并且可以调用到组件类的方法，最后返回结果。例如， `DiskBrakeDecorator` 的 `getModel()` 方法类似于如下所示：

	getModel: function() {
	    return this.component.getModel();
	};

在装饰者类中无需重新定义组件类中存在的方法，只需要按照上面那样操作就可以了。好了，让我们尝试一下新的类结果返回情况：

	var bike = new Bike('Specialized Rock Hopper', 21);
	bike.getPrice()   // -> 210
	
	bike = new DiskBrakeDecorator(bike);
	bike.getPrice()   // -> 260
	bike.getModel()   // -> "Specialized Rock Hopper"

在装饰者方法中通过 `this.component` 可以直接引用到被装饰者对象(组件对象)。如果当一个装饰者定义新的方法时，这些新方法可以通过任意其它装饰器通过包裹对象的方式来进行传递。

	var HornDecorator = new Decorator(Bike, {
	    beepHorn: function(noise) {
	        return noise.toUpperCase();
	    }
	});
	
	var bike = new Bike('Specialized Rock Hopper', 21);
	
	// Let's wrap a HornDecorator with a DiskBrakeDecorator(包裹两个装饰器)
	bike = new HornDecorator(bike);
	bike = new DiskBrakeDecorator(bike);
	
	bike.beepHorn('beep!')    // -> "BEEP!"