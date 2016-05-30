# nightwatchDoc #

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
`--tag`|`-a`| |Filter test modules by tags. Only tests that have the specified tags will be loaded.
`--skiptags`| | |Skips tests that have the specified tag or tags (comma separated).
`--retries`| | |Retries failed or errored testcases up to the specified number of times. Retrying a testcase will also retry the beforeEach and afterEach hooks, if any.
`--suiteRetries`| | |Retries failed or errored testsuites (test modules) up to the specified number of times. Retrying a testsuite will also retry the before and after hooks (in addition to the global beforeEach and afterEach respectively), if any are defined on the testsuite.

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
