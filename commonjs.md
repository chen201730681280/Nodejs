参考https://javascript.ruanyifeng.com/nodejs/module.html



commonjs

为服务器提供的一种模块形式的优化

commonjs模块建议指定一个简单的用于声明模块服务器端的api

入门

commonjs是一个可以复用的js 块，它出口对任何独立代码都起作用的特定对象。

一般包含两个基础的部分：

- exports，包含模块希望提供给其他模块的对象，
- require，模块所需要的可以用来引入和导出其它模块的函数

```js
// define more behaviour we would like to expose
function foobar(){
  this.foo = function(){
    console.log( "Hello foo" );
  }

  this.bar = function(){
    console.log( "Hello bar" );
  }
}

// expose foobar to other modules
exports.foobar = foobar;
```

```js
var foobar = require("./foobar").foobar,
    test   = new foobar();

// Outputs: "Hello bar"
test.bar();
```

### 概述

commonjs规范规定，每个模块内部，module变量代表当前模块。这个变量是一个对象，它的exports属性是对外的接口，加载某个模块，其实是加载该模块的module.exports属性，require方法用于加载模块

#### commonjs模块特点

- 所有代码都运行在模块作用域，不会污染全局作用域
- 模块可以多次加载，但是只会在第一次加载时运行一次，然后运行结果就被缓存了，以后在加载，就直接读取缓存结果。要想模块再次运行，必须清除缓存
- 模块加载的顺序，按照其在代码中出现的顺序

### module对象

Node内部提供一个module构建函数，所有模块都是module的实例，module代表当前模块，有以下属性：

- module.id 模块的识别符，通常是带有绝对路径的模块文件名
- nodule.filename 模块的文件名，带有绝对路径
- module.loaded 返回一个布尔值，表示模块是否已经完成加载
- module.parent 返回一个对象，表示调用该模块的模块
- module.children 返回一个数组，表示该模块要用到的其他模块
- module.exports 表示模块对外输出的值

```json
{ id: '.',
  exports: { '$': [Function] },
  parent: null,
  filename: '/path/to/example.js',
  loaded: false,
  children:
   [ { id: '/path/to/node_modules/jquery/dist/jquery.js',
       exports: [Function],
       parent: [Circular],
       filename: '/path/to/node_modules/jquery/dist/jquery.js',
       loaded: true,
       children: [],
       paths: [Object] } ],
  paths:
   [ '/home/user/deleted/node_modules',
     '/home/user/node_modules',
     '/home/node_modules',
     '/node_modules' ]
}
```

可以通过module.parent判断当前模块是否是入口脚本

```js
if(!module.parent){
	//run with 'node a.js'
    app.listen(8088,function(){
        console.log('app listening on port 8088');
    })
}else{
    module.exports=app;
}
```

### AMD规范和commonjs规范的兼容性

commonjs规范加载模块是同步的，只有加载完成，才能执行后面的操作。AMD规范是同步加载模块，允许指定回调函数。由于nodejs主要用于服务器编程，模块文本一般都已经存在于本地硬盘，所以加载起来比较快，不用考虑非同步加载的方式，所以commonjs规范比较适用。但是，如果是浏览器环境，要从服务区端加载模块，这是就必须采用非同步模式，因此浏览器端一般采用AMD

### requie命令

#### 基本用法

require基本功能是：读入并执行一个js文件，然后返回该模块的exports对象。如果没有发现指定模块，会报错

```js
var example = require('./example.js');
```

#### 加载规则

根据参数的不同格式，`require`命令去不同路径寻找模块文件。

- 如果参数字符串以“/”开头，则表示加载的是一个位于绝对路径的模块文件。比如，`require('/home/marco/foo.js')`将加载`/home/marco/foo.js`。

- 如果参数字符串以“./”开头，则表示加载的是一个位于相对路径（跟当前执行脚本的位置相比）的模块文件。比如，`require('./circle')`将加载当前脚本同一目录的`circle.js`。

- 如果参数字符串不以“./“或”/“开头，则表示加载的是一个默认提供的核心模块（位于Node的系统安装目录中），或者一个位于各级node_modules目录的已安装模块（全局安装或局部安装）。

举例来说，脚本`/home/user/projects/foo.js`执行了`require('bar.js')`命令，Node会依次搜索以下文件。

- /usr/local/lib/node/bar.js
- /home/user/projects/node_modules/bar.js
- /home/user/node_modules/bar.js
- /home/node_modules/bar.js
- /node_modules/bar.js