# nightwatchDoc
nightwatch docment chinese version

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
