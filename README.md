# nightwatchDoc #

## 使用NightWatch ##

### 编写测试 ###

使用合适的CSS选择器来定位页面上的元素，Nightwatch让我们能够非常容易地编写自动化端到端测试。

在我们的项目中创建另外的文件夹用于测试，例如：`tests`。文件夹中的每个文件都会被Nightwatch测试运行期加载为一个测试。一个基本的测试应该看起来像这样：

    module.exports = {
    	'Demo test Google' : function (browser) {
      		browser
      			.url('http://www.google.com')
      			.waitForElementVisible('body', 1000)
      			.setValue('input[type=text]', 'nightwatch')
      			.waitForElementVisible('button[name=btnG]', 1000)
      			.click('button[name=btnG]')
      			.pause(1000)
      			.assert.containsText('#main', 'Night Watch')
      			.end();
    	}
    };
    
一个测试也可以有多个步骤，如果需要的话：

    module.exports = {
    	'step one' : function (browser) {
    		browser
    			.url('http://www.google.com')
    			.waitForElementVisible('body', 1000)
    			.setValue('input[type=text]', 'nightwatch')
    			.waitForElementVisible('button[name=btnG]', 1000)
    	},
    	
    	'step two' : function (browser) {
    		browser
    			.click('button[name=btnG]')
    			.pause(1000)
    			.assert.containsText('#main', 'Night Watch')
    			.end();
    	}
    };
    
测试也可以写成这种格式：

    this.demoTestGoogle = function (browser) {
    	browser
    		.url('http://www.google.com')
    		.waitForElementVisible('body', 1000)
    		.setValue('input[type=text]', 'nightwatch')
    		.waitForElementVisible('button[name=btnG]', 1000)
    		.click('button[name=btnG]')
    		.pause(1000)
    		.assert.containsText('#main', 'The Night Watch')
    		.end();
    };
    
### 使用Xpath ###

Nightwatch也支持xpath选择器。要切换到使用xpath选择，在我们的测试中，可以调用useXpath()方法，下面的例子中就可以看到。要切换回CSS，调用useCss()。
想要一直使用xpath作为默认的选择器策略，在测试设置中设置`"use_xpath": true`。

    this.demoTestGoogle = function (browser) {
    	browser
    		.useXpath() // every selector now must be xpath
    		.click("//tr[@data-recordid]/span[text()='Search Text']")
    		.useCss() // we're back to CSS now
    		.setValue('input[type=text]', 'nightwatch')
    };

### BDD 期望断言 ###
从0.7版本开始，Nightwatch介绍了一种新的BDD风格的断言库，极大地提升了断言的灵活性和可读性。
`expert`断言使用Chai框架里的`Expert`api的一个子集，当前只能用于元素上。下面是一个例子：

    module.exports = {
    	'Demo test Google' : function (client) {
    		client
    			.url('http://google.no')
    			.pause(1000);
    	    // expect element  to be present in 1000ms
    		client.expect.element('body').to.be.present.before(1000);
    		
    		// expect element <#lst-ib> to have css property 'display'
    		client.expect.element('#lst-ib').to.have.css('display');
    		
    		// expect element  to have attribute 'class' which contains text 'vasq'
    		client.expect.element('body').to.have.attribute('class').which.contains('vasq');
    		
    		// expect element <#lst-ib> to be an input tag
    		client.expect.element('#lst-ib').to.be.an('input');
    		
    		// expect element <#lst-ib> to be visible
    		client.expect.element('#lst-ib').to.be.visible;
    		
    		client.end();
    	}
    };
    
`expect`接口提供了一种更灵活和流畅的语言来定义断言，极大地提升了现有的`assert`断言。唯一美中不足就是不能再继续链式断言，并且现在还不支持自定义消息。

要查看可用的`expect`断言，查看[API文档](http://nightwatchjs.org/api/#expect)。

### 测试钩子 ###

Nightwatch 提供了标准的`before`/`after`和`beforeEach`/`afterEach`钩子，让我们在测试中使用。

`before`和`after`会分别在测试套执行之前和之后运行，而`beforeEach`和`afterEach`则分别在每个测试步骤之前和之后执行。

这些方法都以Nightwatch实例作为参数。

示例：

    module.exports = {
    	before : function(browser) {
    		console.log('Setting up...');
    	},
    	
    	after : function(browser) {
    		console.log('Closing down...');
    	},
    	
    	beforeEach : function(browser) {
    	
    	},
    	
    	afterEach : function() {
    	
    	},
    	
    	'step one' : function (browser) {
    		browser
    		// ...
    	},
    	
    	'step two' : function (browser) {
    		browser
    		// ...
    		.end();
    	}
    };
	
上面的例子中，方法的调用顺序如下：

`before(), beforeEach(), "step one", afterEach(), beforeEach(), "step two", afterEach(), after()`

### 异步测试钩子 ###

所有的`before[Each]`和`after[Each]`也都可以进行异步操作，这时候就需要传递`callback`作为第二个参数。
**在异步操作完成时，必须调用`done`函数作为最后一步，否则会引起超时错误。**

#### beforeEach & afterEach示例 ####

    module.exports = {
    	beforeEach: function(browser, done) {
    		// performing an async operation
    		setTimeout(function() {
    	    	// finished async duties
    			done();
    		}, 100);
    	},
    	
    	afterEach: function(browser, done) {
    		// performing an async operation
    		setTimeout(function() {
      			// finished async duties
      			done();
    			}, 200);
    	}
    };

#### 控制`done`调用超时 ####

默认情况下，`done`调用超时时间设置在10秒（单元测试是2秒）。在某些情况下这个时间可能不够，为了避免出现超时错误，我们可以提高超时时间，只要在我们的外部全局文件中定义一个`asyncHookTimeout`属性（以毫秒为单位）。

例如，参考提供的[globalModule](https://github.com/nightwatchjs/nightwatch/blob/master/examples/globalsModule.js#L20)示例。

#### 显式地将测试置失败 ####

要在测试钩子里内部将测试置失败，只要简单地使用`Error`参数来调用`done`即可。

    module.exports = {
    	afterEach: function(browser, done) {
    		// performing an async operation
    		performAsync(function(err) {
      			if (err) {
        			done(err);
      			}
      			// ...
    		});
    	}
    };

### 外部全局变量 ###

大多数情况下，将我们的全局变量定义在一个外部文件中会更有用。这要求我们在globals_path属性中指定，而不是在nightwatch.json中定义。

我们还可以按需为每个测试环境覆盖全局变量。比如说我们要在本地和远程机器上运行测试。大多数时候我们会需要不同的设置

#### 全局钩子 ####

用于测试套的钩子同样可以全局使用，可以查看下面的例子。如果是全局钩子，那么`beforeEach`和`afterEach`关联的就是一个测试套，他们分别在测试套之前和之后运行。

#### 全局设定 ####

有不少全局变量保存了测试的设置，这些设置能够控制测试执行方式。这些内容在示例[globalModule](https://github.com/nightwatchjs/nightwatch/blob/master/examples/globalsModule.js)中有详细细节。

示例：

    module.exports = {
    	'default' : {
    		isLocal : true,
    	},
    	
    	'integration' : {
    		isLocal : false
    	},
    	
    	// External before hook is ran at the beginning of the tests run, before creating the Selenium session
    	before: function(done) {
    		// run this only for the local-env
    		if (this.isLocal) {
      			// start the local server
      			App.startServer(function() {
        			// server listening
        			done();
      			});
      		} else {
      			done();
      		}
      	},
      	
      	// External after hook is ran at the very end of the tests run, after closing the Selenium session
      	after: function(done) {
      		// run this only for the local-env
    		if (this.isLocal) {
      			// start the local server
      			App.stopServer(function() {
        			// shutting down
        			done();
      			});
    		} else {
      			done();
    		}
    	},
    	
    	// This will be run before each test suite is started
    	beforeEach: function(browser, done) {
    		// getting the session info
    		browser.status(function(result) {
      			console.log(result.value);
      			done();
    		});
    	},
    	
    	// This will be run after each test suite is finished
    	afterEach: function(browser, done) {
    		console.log(browser.currentTest);
    		done();
    	}
    };


## 运行测试##

### 测试运行器 ###

Nightwatch包含了一个命令行测试运行器，它让我们能够更容易地运行测试和生成有用的报告。根据我们的安装类型不同，它有一些不同的选项来使用测试运行器。

#### 全局 ####

如果是全局安装的Nightwatch(使用`-g`选项)，那么二进制的`nightwatch`可以在任何地方使用：

    $ nightwatch [source] [options]

#### 项目级别 ####

如果是将Nightwatch作为项目的一个依赖来安装的，那么可以从`node-modules/.bin`文件夹引用执行文件：

##### Linux和MacOSX: #####

    $ ./node_modules/.bin/nightwatch [source] [options]

##### Windows: #####

创建一个文件`nightwatch.js`，然后在文件中添加如下代码：

    require('nightwatch/bin/runner.js');

然后如下运行：

    C:\workspace\project> node nightwatch.js [source] [options]

#### 测试资源 ####

可选的`optional`参数可以是一个文件，也可以是多个文件，或者是一整个文件夹。这样可以无视`src_folders`设置。

##### 示例 - 单个测试 #####

    $ nightwatch tests/one/firstTest.js

##### 示例 - 2个测试 #####
    
    $ nightwatch tests/one/firstTest.js tests/secondTest.js

##### 示例 - 1个单独的测试以及一个文件夹 #####

    $ nightwatch tests/one/test.js tests/utils

### 命令行选项 ###

测试运行器支持很多运行时选项。要查看所有选项，运行如下命令：

    $ nightwatch --help

**Option Name** | **Shortname** | **Default** | **Description** 
:------|:------:|------|:------
 `--config` | `-c` | `.nightwatch.json` | `nightwatch.json`文件的位置——运行器使用的配置文件，其中也包含了Selenium WebDriver选项。
 `--output`|`-o`|`tests_output`|JUnit XML报告保存的位置。
`--reporter`|`-r`|`junit`|预定义报告器的名称 或 准备使用的自定义报告文件的路径。
`--env`|`-e`|`default`|使用哪个测试环境——定义在`nightwatch.json`中。
`--verbose`| | |在会话过程中显示扩展的selenium命令日志。
`--version`|`-v`| |显示版本号。
`--test`|`-t`| |仅运行指定的测试。默认它会尝试运行文件夹及其子文件夹中的所有测试。
`--testcase`| | |只能和--test一起使用。	运行当前suite/module中指定的测试用例。
`--group`|`-g`| |仅运行指定的测试组
`--skipgroup`|`-s`| |跳过一个或多个测试组。
`--filter`|`-f`| |指定一个过滤器作为文件名格式,加载测试文件时仅加载符合要求的文件。
`--tag`|`-a`| |通过标签过滤测试模块。只有指定此标签的测试会被加载。
`--skiptags`| | |跳过指定标签的测试（多个标签以comma分隔)。
`--retries`| | |重试失败或错误的测试用例最多指定次数。
`--suiteRetries`| | |重试失败或错误的测试套最多指定次数。

## 扩展NightWatch
### 自定义命令
大多数时候我们都会需要扩展NigthWatch命令来适应我们自己的应用需求，而我们也只需要创建一个文件夹，然后在这个文件夹里定义我们的命令，每个命令对应一个文件。
然后，在nightwatch.json文件中，将这个文件夹的路径指定给custom_commands_path属性。
命令的名称就是文件名，并且它需要遵循如下模式：

```
exports.command = function(file, callback) {
	var self = this, imageData, fs = require('fs');
	
	try {
    	var originalData = fs.readFileSync(file);
    	var base64Image = new Buffer(originalData, 'binary')
    	.toString('base64');
    	imageData = 'data:image/jpeg;base64,' + base64Image;
    } catch (err) {
    	console.log(err);
    	throw "Unable to open file: " + file;
  	}
  	
  	this.execute(
  		function(data) { // execute application specific code
  			App.resizePicture(data);
  			return true;
  	},
  	
  	[imageData], // arguments array to be passed
  	
  	function(result) {
  		if (typeof callback === "function") {
  			callback.call(self, result);
  		}
  	}
  	);
  	
  	return this; // allows the command to be chained.
};
```
下面的例子定义了一个命令（如：resizePicture.js），它加载一个图片文件作为data-URI，并且调用了一个定义在应用内部的resizePicture方法。
通过这个命令，测试文件会像这样：
```
module.exports = {
  "testing resize picture" : function (browser) {
    browser
      .url("http://app.host")
      .waitForElementVisible("body")
      .resizePicture("/var/www/pics/moon.jpg")
      .assert.element(".container .picture-large")
      .end();
  }
};
```
###自定义断言
