---
layout: post
title: handlerbars的使用及模板预编译
category: web前端
tags: web
keywords: handlerbars
description:
---

-   handlebars介绍
-   handlebars的用法
-   handlebars预编译


##  [handlebars](http://handlebarsjs.com) 介绍
---

handlebars是一款前端模板引擎，使用模板引擎的作用是，可以解决通过js拼接html的方式渲染页面的缺点。

主页：[handlebars](http://handlebarsjs.com)

##   quick example

````js

//导入库
<script src="../bower_components/jquery/jquery.min.js" type="text/javascript"></script>
<script src="../bower_components/handlebars/handlebars.js" type="text/javascript" charset="utf-8" ></script>

````

````js

<script id="demoTpl" type="text-x-handlebars-template">
	  {{#each this}}
        <div>
         <h1>{{title}}</h1>
         <p> {{content}}</p>
        </div>
      {{/each}}
</script>

````

````js

   //示例数据
	var data = [
		{ "title":"aaa","content":"contentaaa"},
		{ "title":"aaa","content":"contentaaa"},
		{ "title":"aaa","content":"<h1>h1</h1>"},
		{ "title":"aaa","content":"contentaaa"}
	];

    //jquery初始化完成
	$(function(){
         //编译模板
		 var demoTpl = Handlebars.compile($("#demoTpl").html());
		 //获取渲染后的html内容
		 var html = demoTpl(data);
		 //渲染dom，目标为#content
		 $("#content").html(html)
	});

````


##  handlebars的用法
---

### 显示模板中的html 

若数据中有html，如： ```` { "title":"aaa","content":"<h1>h1</h1>"} ```` 则模板绑定{{}}时会显示 ```` "<h1>h1</h1>" ```` 内容
使用三对大括号{{{ }}} 可以显示 html效果

###	Block表达式

有时候当你需要对某条表达式进行更深入的操作时，Blocks就派上用场了，在Handlebars中，你可以在表达式后面跟随一个#号来表示Blocks，然后通过{{/表达式}}来结束Blocks。
如果当前的表达式是一个数组，则Handlebars会“自动展开数组”，并将Blocks的上下文设为数组中的元素。

````
<ul>  
{{#programme}}
    <li>{{language}}</li>
{{/programme}}
</ul>  

````


````
//使用数据
{
  programme: [
    {language: "JavaScript"},
    {language: "HTML"},
    {language: "CSS"}
  ]
}

//结果
<ul>  
  <li>JavaScript</li>
  <li>HTML</li>
  <li>CSS</li>
</ul>  


````

###	遍历

````
	  {{#each this}}
        <div>
         <h1>{{title}}</h1>
         <p> {{content}}</p>
        </div>
      {{/each}}
````


考虑到这样一种情况，data中的数据直接就是数组，没有key。如：

````js

	var data1 = [
		 "a","b","c","d","e"
	];

````

那么模板怎么写？

答案是，用this关键字

````js

	  {{#each this}}
        <div>
         <h1>{{this}}</h1>
        </div>
      {{/each}}


````


### if else,unless，with

````
{{#if list}}
<ul id="list">  
    {{#each list}}
        <li>{{this}}</li>
    {{/each}}
</ul>  
{{else}}
    <p>{{error}}</p>
{{/if}}
````

unless和if意思正好相反，语法使用是相同的

with是判断属性是否存在，存在则绑定数据，不存在则不绑定

###  注释

{{! handlebars comments }}

###  Path

````
. :子属性
../ :父属性


###  自定义helper

这个方法用来注册一些handlebars静态工具方法。


````js

//定义
Handlebars.registerHelper('methodName', function(para1,para2){
		//做的你工作
});

//调用
{{methodName para1 para2}}

````

一次性也可以注入多个help

````
andlebars.registerHelper({
    fun1: function() {},
    fun2: function() {}
});
````

### hash参数

在helper方法内部，可以通过option.hash获取到方法的上下文参数

````js

Handlebars.registerHelper('list', function(items, options) {
  var out = '<ul>';
  for(var i=0, l=items.length; i<l; i++) {
    var item = options.fn(items[i]);
    out = out + '<li class="'+options.hash.class+'">' + item + '</li>';
  }
  return out + '</ul>';
});


````

一个判断奇数偶数的例子

````js

//判断是否是偶数
Handlebars.registerHelper('if_even', function(value, options) {
  console.log('value:', value); // value: 2
  console.log('this:', this); // this: Object {num: 2}
  console.log('fn(this):', options.fn(this)); // fn(this): 2是偶数
  if((value % 2) == 0) {
    return options.fn(this);
  } else {
    return options.inverse(this);
  }
});

````




##  handlebars预编译
---

handlebars预编译依赖于handlebars插件，安装方式：

```` sudo npm install handlebars -g  ````

我们有的时候希望能把handlebars的模板文件单独抽取出来管理，可以写一个后缀为handlebars的文件，然后把模板写到文件中，例如：

````

  {{#each this}}
    <div>
     <h1>{{title}}</h1>
     <p> {{content}}</p>
    </div>
  {{/each}}


````

之后我们通过命令，模板文件编译为handlebars js文件

```` handlebars demoList.handlebars -f  demoTpl.js ````

**这里顺便说一句，handlebars后缀名很长，我试过换一个简单的后缀，结果发现编译后就找不到模板了。所以后缀还是不能随便换**

执行后会在本文件夹生成一个js文件，接着我们在页面引入这个js

````js
//引入生成的js
<script src="../component/Tpl/demoList.js"></script>
````

最后将模板渲染到页面

````js

	var data = [
		{ "title":"aaa","content":"contentaaa"},
		{ "title":"aaa","content":"contentaaa"},
		{ "title":"aaa","content":"contentaaa"},
		{ "title":"aaa","content":"contentaaa"}
	];

	$(function(){
        //使用预编译的tpl ，demoList就是我们之前生成的模板js的文件名
        var html = Handlebars.templates.demoList(data)
        $("#content").html(html);
	});

````



##  demo

本文示例demo见 [demo-web](https://github.com/coolnameismy/demo-web)


##  参考和其他资料

-   [handlebars实用教程](http://www.cnblogs.com/iyangyuan/archive/2013/12/12/3471227.html)
-   [极客标签视频教程](http://www.gbtags.com/gb/gbliblist/7.htm)
-   [Handlebars.js 模板引擎](http://caibaojian.com/handlebars-js.html)

## 最后

如果大家喜欢，请[github上follow和star](https://github.com/coolnameismy)