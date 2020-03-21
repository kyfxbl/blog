title: chrome extensions
date: 2013-09-24 11:22
categories: 其它 
---
![chrome](http://pic.kyfxbl.com/a40.jpg)
chrome的扩展（extensions），和插件（plugins）相比，能做的事情是比较有限的
<!--more-->

# 能力范围

extensions基本上可以做2类事情： 

1、与原始页面的内容交互，比如获取DOM里的内容，注入javascript脚本执行等 

2、与浏览器交互，比如操作chrome的windows、tabs，访问chrome的书签、历史记录等 

做相应的操作，需要在manifest文件里声明权限，在用户安装的时候，会提示用户，由用户决定是否允许（跟android app是一个意思） 

范围基本局限在上述2点，像访问本地硬盘、调用本地应用等，还是不支持的。如果要做本地应用的一些事情，用extension是做不到的，必须用plugin，基于NPAPI 

# 扩展的形式

扩展主要有以下几种形式： 

## browser action或page action 

这种形式的扩展会在chrome地址栏（omnibox）那里增加一个按钮，允许用户点击 

然后可以弹出一个popup窗口，在popup html里引用的javascript，操作的就是popup的DOM，而不是原始页面的DOM

```
function click(e) {
  chrome.tabs.executeScript(null, {code:"document.body.style.backgroundColor='" + e.target.id + "'"});
  window.close();
}

document.addEventListener('DOMContentLoaded', function() {
  var divs = document.querySelectorAll('div');
  for (var i = 0; i < divs.length; i++) {
    divs[i].addEventListener('click', click);
  }
});
```
如上述的代码，document、window，都是popup里面的DOM元素，而不是原始页面里的元素 

## background javascript 

也可以不设置action按钮，通过background加载长期后台运行的javascript脚本，做一些操作，比如在后台运行一个定时任务等 

## content script 

扩展还可以定义一系列content script脚本，注入到原始页面里执行 

content script是在一个特殊环境中运行的，这个环境叫做isolated world（隔离环境）。它们可以访问所注入页面的DOM，但是不能访问里面的任何javascript变量和函数。对每个content script来说，就像除了它自己之外再没有其它脚本在运行；反过来也是成立的：页面里的javascript也不能访问content script中的任何变量和函数 

隔离环境使得content script可以修改它的javascript环境而不必担心会与这个页面上的其它content script冲突。例如，一个content script可以包含jquery v1，而页面可以包含jquery v2，它们之间不会产生冲突 

另一个重要的优点是隔离环境可以将页面上的脚本与扩展中的脚本完全隔离开。这使得开发者可以在content script中提供更多的功能，而不让web页面利用它们 

尽管content script的执行环境与所在的页面是隔离的，但它们还是共享了页面的DOM。如果页面需要与content script通信（或者通过content script与extension通信），就必须通过这个共享的DOM 

关于chrome extensions的概述，没有比官方overview更好的文章： 

英文版：[http://developer.chrome.com/extensions/overview.html](http://developer.chrome.com/extensions/overview.html) 
360翻译中文版：[http://open.chrome.360.cn/extension_dev/overview.html](http://open.chrome.360.cn/extension_dev/overview.html)

# 例子

作为练手，昨晚开发了一个chrome的扩展。效果是在页面上选中文本，然后在右键菜单里点击按钮，保存到localStorage里，在popup window里可以看到保存的笔记

数据是保存在localStorage里的，由于localStorage只能保存String，但是应用的数据结构需要用到object，所以要用JSON的方法转化一下，详见代码 

![](http://dl.iteye.com/upload/attachment/0081/5085/2c9c8ec5-6292-3533-8187-07c0f634f05b.png)

扩展用到了background javascript，用来生成右键菜单、处理点击；以及一个popup页面，用来展示保存的笔记；由于没有和页面的直接交互，就不需要用到content script 

首先是manifest
```
{
  "manifest_version": 2,
  "name": "kyfxbl note",
  "description": "This extension notes text content from page",
  "version": "1.4",
  "background": {
    "scripts": ["common.js","uuid.js","background.js"]
  },
  "browser_action": {
    "default_icon": "icon.png",
    "default_popup": "popup.html",
    "default_title": "kyfxbl note"
  },
  "permissions": [
    "tabs",
    "contextMenus",
    "http://*/*"
  ]
}
```

然后是background.js，在chrome启动时就会在后台运行

```
initLocalStorage();

// see http://developer.chrome.com/extensions/contextMenus.html for details
var createMenuProp = {
	"title" : "note",
	"contexts" : [ "selection" ],
	"onclick" : noteIt
};

chrome.contextMenus.create(createMenuProp);

function noteIt(info, tab) {
	var uuid = Math.uuid(16);
	var note = new Note(info.selectionText);
	var object = JSON.parse(localStorage.mynotes);
	object[uuid] = note;
	localStorage.mynotes = JSON.stringify(object);
}
```

最后是popup.js，点击时显示笔记

```
<!doctype html>
<html>

<head>
    <link type="text/css" rel="stylesheet" href="popup.css" />
    <script src="jquery-1.8.0.js"></script>
    <script src="common.js"></script>
    <script src="popup.js"></script>
    <title>kyfxbl note</title>
</head>

<body>
	<div id="wrapper"></div>
	<button id="btn_clear">clear</button>
</body>

</html>
```

```
$(document).ready(function() {

	$("#btn_clear").click(clearLocalStorage);

	renderNotes();

	function renderNotes() {

		$("#wrapper").empty();// 先清空页面

		var notes = JSON.parse(localStorage.mynotes);

		$.each(notes, function(index, value) {

			var $div = $("<div>");
			var content = value.content;
			var $content = $("<span class='content'>" + content + "</span>");
			var $uuid = $("<span class='uuid'>" + index + "</span>");
			var $button = $("<button>delete</button>");
			$button.click(deleteCurrentNote);

			$div.append($content);
			$div.append($uuid);
			$div.append($button);
			$("#wrapper").append($div);

		});
	}

	function clearLocalStorage() {
		localStorage.clear();
		initLocalStorage();
		renderNotes();
	}

	function deleteCurrentNote() {
		var uuid = $(this).prev().text();
		var object = JSON.parse(localStorage.mynotes);
		delete object[uuid];
		localStorage.mynotes = JSON.stringify(object);
		renderNotes();
	}

});
```

# 学习方法

感觉google出的东西都比较好学，因为它的架构很清晰，相关的文档比较全，API设计得也比较好 

当学习一个新的框架或者平台的时候，可以按三步走： 

1、了解场景和边界，即这个东西是用在什么场合，可以解决什么问题，不能解决什么问题 

2、学习架构，在高层面上明白有什么组件，怎么交互。比如android的4种组件，chrome extension的background、browser action、content script的关系 

3、学习API，在上述2个步骤以后，就可以深入到reference里，看看都有什么API，应该怎么调用。每个平台或者框架能做什么事情，最终都是由API来体现的