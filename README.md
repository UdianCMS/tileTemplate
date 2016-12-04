# tileTemplate 

A simple, high performance Javascript template engine.

一个简单的、高性能的 Javascript 模板引擎。 

![](http://pandao.github.io/tiletemplate/tests/test-speed.png)

> 注：测试结果会因环境而有所不同，仅供参考。

### 主要特性

- 简单小巧，精简后只有 `5K`，开启 gzip 后只有 `2.4K`；
- 原生语法，高性能预编译和渲染模板 [（性能测试）](http://pandao.github.io/tiletemplate/tests/test-speed.html "(性能测试)")；
- 安全机制，过滤和转义危险语句[（安全测试）](http://pandao.github.io/tiletemplate/tests/test-xss-filter.html "(安全测试)")；
- 支持各种前端模块化加载器（CommonJS / AMD / CMD 等）[( Require.js示例 ](http://pandao.github.io/tiletemplate/examples/requirejs-test.html) 、[Sea.js示例 )](http://pandao.github.io/tiletemplate/examples/seajs-test.html)；
- 支持在 `Node.js` 环境下运行（`v1.6.0 起支持预编译模板文件为 CMD 模块`），同时也支持 `Express.js`；
- 支持调试，精确定位并通过控制台输出和显示错误或异常信息（[查看调试](http://pandao.github.io/tiletemplate/tests/test-debug.html)）；
- 支持所有主流的浏览器（`IE6+`）；
- 支持 `include` 和自定义标签语法；

### 下载和安装

- [源码   tiletemplate.js](https://github.com/pandao/tileTemplate/tree/master/src/tiletemplate.js "源码")
- [压缩版 tiletemplate.min.js](https://github.com/pandao/tileTemplate/tree/master/dist/tiletemplate.min.js "压缩版")

通过 NPM 安装：

```shell
npm install tiletemplate
```

通过 Bower 安装：

```shell
bower install tiletemplate
```

### 使用方法

编写模板：

```html
<!-- type可以任意定义 text/xxxx -->
<script id="test-tpl" type="text/tileTemplate">
    <h1><%=title%></h1>
    <ul> 
        <% for (i = 0, len = list.length; i < len; i ++) { %>
            <li>
				用户: <%=list[i].user%>
				网站：<a href="<%=list[i].site%>"><%=list[i].site%></a>
			</li>
        <% } %>
    </ul>
</script>
```

预编译模板：

```javascript
var data = {
    title : "标题XXX",
    list  : []
}; 

// 返回一个函数
var compiler = tileTemplate.compile(document.getElementById('test-tpl').innerHTML, data);
```

渲染模板：

```javascript
var data = {
    title : "标题XXX",
    list : []
}; 

for (var i = 0; i < 10; i ++) {
    data.list.push({
        index: (i+1),
        user: '<strong style="color:red">tileTemplate '+(i+1)+'</strong>',
        site: 'https://github.com/pandao/tileTemplate'
    });	
}

// 输出HTML
// document.getElementById('output').innerHTML = compiler(data);
document.getElementById('output3').innerHTML   = tileTemplate.render("test-tpl", data);
```

> 注：同时也支持在 [Require.js](http://pandao.github.io/tiletemplate/examples/requirejs-test.html) 和 [Sea.js](http://pandao.github.io/tiletemplate/examples/seajs-test.html) 中使用。

### 在 Node.js 使用：

```javascript
var tileTemplate = require("../src/tiletemplate.node");

// 通过 NPM 安装的
// var tileTemplate = require('tiletemplate');

// 设置基本目录
tileTemplate.config("basePath", __dirname + "/tpl/");

// tileTemplate.render(文件名/模板内容, 数据, 编码);
// console.log(tileTemplate.render("Hello <%=str%>", {str:"wolrd!"}));

// 预编译某个模板，用于循环渲染
//var compiler = tileTemplate.compile(tileTemplate.readFile("list"), {list : [...]});

// v1.5.0 版本起无需填写扩展名，默认为 tile.html，可另行配置
var html = tileTemplate.render("test", data);
var http = require('http');

http.createServer(function (request, response) {
    response.writeHead(200, {'Content-Type': 'text/html'});
    response.end(html);
}).listen(8888);

console.log('Server running at http://127.0.0.1:8888/');
```

> 注：`tileTemplate.readFile(文件名, 编码)` 方法只能在 `Node.js` 下使用。

### 在 Express.js 中使用

```javascript
var express		 = require('express');
var app			 = express(); 
var tileTemplate = require("tiletemplate");

// 初始化Express支持
tileTemplate.expressInit(app, __dirname + "/tpl/");

app.get('/', function (req, res) { 
	res.render('index', data);   // v1.5.0 版本起无需填写扩展名，默认为 tile.html，可另行配置
});

var server = app.listen(3000, function() {
    console.log('Listening on port %d', server.address().port);
});
```

### 主要语法

`tileTemplate` 目前只支持原生语法。

文本输出：

```html
<%=变量%>
<img src="<%=avatar%>" />
```

JS语句：

```html
<% if (list.length > 0) { %>
<p>Total: <%=list.length%> </p>
<% } else { %>
<p>暂时没有</p>	
<% } %>

<% var total = list.length; %>

<%=(list.index>1?'>1':'<1')%>
...
```

变量注释：

```html
<%=#变量%>
```

行注释：

```html
//注释文本
//<%=(list.index>1?'>1':'<1')%>
<% /* 注释文本 */ %>
<!-- HTML式注释 -->
```

嵌套模板（支持多级嵌套）：

```html
<% include('模板id') %>
```

转义字符（默认不转义字符，需要的在前面加上@）：

```html
<img src="<%=@avatar%>" />
```

> 作用：过滤和防止 XSS 攻击。例如：当 `avatar` 的值为 `http://xxxx/xx.jpg" onload="alert(123)`。

自定义标签语句：

```javascript
// 定义标签语句
tileTemplate.tag("em", function(content) {         
    if(content == 12) {
		var img = "http://img.t.sinajs.cn/t4/appstyle/expression/ext/normal/0b/tootha_thumb.gif";
        return '<img src="'+img+'" alt="em'+content+'"/>';
    } else {
        return content.toString();
    }
});

tileTemplate.tag("time", function() {
    return " time: " + (new Date).getTime();
}); 

/*
// 使用标签语句
<%=tag:em:12%>
<%=tag:em:haha%>
<%=tag:em:哈哈%>    
<%=tag:time%>
*/
```

> 注：自定义标签语句只能输出字符串。

### 主要方法

默认选项：

```javascript
{
    debug    : false,  // 是否开启调试功能，默认不开启，在生产环境下，这样能获得更高的性能和速度；开发时，请开启；
    cached   : true,   // 是否开启缓存，默认开启，性能更好
    filter   : true,   // 是否过滤模板中的危险语句等，如alert等
    openTag  : "<%",   // 模板开始标签
    closeTag : "%>"    // 模板结束标签
}
```

修改和设置配置选项：

```javascript
// 使用set或config方法修改配置选项，config为别名
// 批量设置
tileTemplate.set({
    openTag : "{{",
    closeTag : "}}"
});

tileTemplate.config({
    openTag : "{{",
    closeTag : "}}"
});

// 单个设置
tileTemplate.set("openTag", "{{");
tileTemplate.config("openTag", "{{");
```

渲染模板：

```javascript
/**
 * @param  {String}           id            模板的 ID，或者直接传入模板内容
 * @param  {Object}           data          传入模板的数据
 * @param  {String}           filename      当不通过ID获取模板，而是直接传入模板，需要设置一个模板名称
 * @param  {Boolean}          isModule      是否预编译为 CMD 模块
 * @return {Function|String}  html          返回模板渲染后的 HTML 或预编译模块的字符串
 */

tileTemplate.render(id, data, filename, isModule);
```

预编译模板：

```javascript
/**
 * @param  {String}           tpl      模板的内容
 * @param  {Object}           data     传入模板的数据，默认为 {}
 * @param  {Object}           options  配置选项，参数为：include 是否为嵌套的模板，name 模板ID，isModule 是否预编译为 CMD 模块
 * @return {Function|String}  fn       返回预编译函数或预编译模块的字符串
 */

tileTemplate.compile(tpl, data, options);
```

自定义标签语句：

```javascript
/**
 * @param  {String}     name       标签名称
 * @param  {Function}   callback   处理标签的回调方法，参数为content，代表标签语句传入的参数
 * @return {Void}	    void       无返回值
 */

tileTemplate.tag(name, callback);
```

清除某个模板的缓存：

```javascript
/**
 * @param  {String}     id         模板的ID或者文件名
 * @return {Boolean}    bool       返回 true
 */

tileTemplate.clear(id);
```

扩展 tileTemplate：

```javascript
tileTemplate.xxx = xxxx;

// 或者
tileTemplate.extend({
	xxx : "xxxx"
	methodXXX : function() {
	   //...
	}
});
```

### 更新日志

[查看更新日志](https://github.com/pandao/tileTemplate/blob/master/CHANGE.md)

### License

The MIT License (MIT)

Copyright (c) 2014-2016 Pandao
