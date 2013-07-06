# Benchmark #

### 概述 ###

**JS.Benchmark** 提供了一个用来测量JavaScript代码执行时间的工具。当进行一个评估测量时，需要为评测提供一个名称，运行次数及要执行的函数体：

	JS.Benchmark.measure('String#join', 20, {
	    test: function() {
	        ['a', 'list', 'of', 'strings'].join(' ');
	    }
	});

**JS.Benchmark** 将针对 `test` 方法，按照指定的次数运行它，然后通过 **console** 模块打印出函数执行时间的平均值和标准差值。

如果在评测过程中需要添加一些设置或初始化等操作，而同时又不希望将这些设置或初始化执行时间包括在评测时间内时，那么为了保证得到真正需要的评测，需要将这些相应操作置于它们自己所属的函数中。在 `setup` 和 `test` 函数中状态可以通过将值赋给 **this** 来达到共享：

	JS.Benchmark.measure('Module#ancestors', 10, {
	    setup: function() {
	        this.module = new JS.Module({ include: [JS.Comparable, JS.Enumerable] });
	    },
	    test: function() {
	        this.module.ancestors();
	    }
	});

结果，上面通过 **Benchmark** 评测报告的时间中是不包括运行 `setup` 函数的时间的，仅包含 `test` 函数执行时间。