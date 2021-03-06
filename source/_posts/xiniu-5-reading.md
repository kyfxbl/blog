title: 犀牛书第五版读书笔记
date: 2013-09-24 10:29
categories: javascript
---
![xiniu](http://pic.kyfxbl.com/xiniushu.jpg)
犀牛书第五版的读书笔记
<!--more-->

# chapter 2

javascript采用Unicode字符集，所以理论上是可以用中文编程的，虽然没有人这么做 

区分大小写，但其主要所处的环境HTML，则是不区分大小写的 

语句最后的;是可选的，但是一般来说，强烈建议写上;

注释写法和java完全一样 

最重要的字面量是[]和{}，前者是数组，后者是对象 

标识符的规则和java也一样。另外有一个不成文的规定：通常用\_\_name\_\_这种形式，表示对象的私有属性 

java中把类的成员变量称为字段（field），而在javascript中称为属性（property）

# chapter 3

javascript中的数据类型分为基本数据类型和引用数据类型 

基本数据类型包括number, string, boolean, null, undefined 

引用数据类型包括object, function, array 

有一些特殊的number，包括NaN, Infinity, Nunber.MAX_VALUE等，它们typeof的值都是number，它们的constructor是function Number(){}
 
string是不可变的，这点和java中一样 

定义函数有3种形式：
```
function func(){
    // logic
}
```
```
var func = function(){
    // logic
};
```
```
var func = new Function();
```
最常见的是第一种形式

javascript中的Object与java中有很大不同，实际上它只是一个键值对而已，可以理解成Map这种数据结构。定义Object也有两种方式：

<pre>
var o = new Object();
var o = {};
</pre>

由于Object只是键值对，所以创建了Object之后，可以任意增加其属性：

```
o.age=23;
a.name="kitty";
```

array的typeof值是object，constructor是function Array(){}，所以实际上它是一个object。但由于array很重要，所以通常也单独作为一种数据类型。定义array也有两种方式：

```
var a = new Array();
var a = {};
```
这两种方式都很常见，后一种用得更多一些 

javascript中的array是弱类型的，可以将任意类型放入数组中，而不像java中的数组那样，只能放同一种类型 

null的typeof值是object，但它没有constructor属性，null没有任何属性 

null在boolean环境当做false使用，在数字环境当做0使用，在string环境当做"null"使用 

undefined的typeof值是object，但它没有constructor属性，undefined没有任何属性 

undefined在boolean环境当做false使用，在数字环境当做NaN使用，在string环境当做"undefined"使用 

number, string, boolean这三种基本数据类型都有对应的包装类

# chapter 4

javascript中的变量范围只有2种：全局变量和函数局部变量，不存在block scope

变量有原始类型和引用类型的区别。原始类型的赋值是值复制，引用类型的赋值是引用复制

javascript中也有垃圾回收机制
```
var s = "hello";          // Allocate memory for a string
var u = s.toUpperCase( );  // Create a new string
s = u;                    // Overwrite reference to original string
```
上述代码执行完毕以后，字符串"hello"成为不可触及的，稍后会被垃圾回收机制释放空间。垃圾回收机制和闭包有很大关系

javascript解释器启动后，在执行任何javascript代码之前，它首先创建一个全局对象（global object）。所有的全局变量和函数外部定义的function，都成为它的属性和方法

当一个函数被调用时，则创建一个调用对象（call object）。所有的局部变量和函数参数，都成为它的属性，嵌套函数则成为它的方法。调用对象的声明周期比全局对象短，但起到的作用是一样的 

结合4和5，可以得知，javascript中所有的变量和函数，其实都是某个对象的属性(property)和方法(method)

每次当javascript解释器开始执行一个函数后，它为函数创建一个新的执行上下文（execution context）

每个execution context都有一个关联的scope chain。scope chain是一个对象列表，当javascript查找变量时，就自底向上进行查询。如果是最外层的code，则其scope chain上只有一个global object。如果是最外层的function，则其scope chain上有2个对象，先是call object（包含局部变量和参数），然后是global object。内层嵌套的function，则有3个对象，第一个是自身的call object，第二个是外层function的call object，最后才是global object。由于这个机制，所以代码可以访问到外围的变量，不能访问到内部嵌套的变量

# chapter 5

经javascript解释器能计算出一个有效值的语句，即javascript表达式，比如17,"hello",i,i+3等，都是表达式 

操作符==在可能的情况下，会自动进行类型转换。===则不会 

# chapter 6

javascript中的switch比java中的好用，不仅可以用来检查number，也可以直接用来匹配string 

for/in语句
```
for (var prop in object) {
    alert(prop);
}
```

with的作用是把对象临时地置于scope chain的第一位，在执行完毕后则清除。with语句的性能比较差，如果没有充分的理由，最好少用

# chapter 7

javascript中的对象只是键值对而已，创建object有2种方法：
```
var o = {};
var o = new Object();
```

访问对象属性也有2种办法：
```
object.prop
object[prop]
```

通常使用前者，但是在对象属性名在运行时才能确定的情况，只能使用后者 

用push()和pop()方法，可以使array具有First In Last Out的行为。shift()和unshift()方法则相反，是在array的头部进行操作

# chapter 8

function定义时指定的参数数量可以和实际调用时传递的参数数量不同，没有传递的参数设为undefined，多余的参数被忽略，但是可以通过arguments属性来访问，这是call object的属性 

没有return语句的函数，以及直接return的函数，返回值都是undefined 

arugments有一个callee属性，它指向function自身 

javascript中的function也是对象，可以赋值给其它对象作为属性。而且也有自己的属性，比如很重要的prototype属性 

每个函数都有一个prototype属性，该属性指向一个预先定义的原型对象。当函数和new操作符结合使用时，prototype对象的所有属性都会映射到新创建的对象上。所以prototype属性在创建新对象时，起到很重大的作用 

function还有call()和apply()方法，这2个方法作用是一样的，只是call()方法接受多个参数，而apply()接受所有参数组成的数组。这2个方法的第一个参数，作为this的值 

function中的this不是由如何定义决定的，而是由function如何被调用决定的。用object.function()的形式调用时，this就指向object。如果用function.call()或者function.apply()方式调用时，this指向第一个参数。如果用new function()形式调用时，此时function是作为一个构造函数，this指向新创建的对象 

javascript中的function是文法作用域，而不是动态作用域。这意味着函数是在定义的范围内运行，而不是执行的范围内运行。当一个函数被定义时，它当前的scope chain就被保存，并成为函数的内部状态而固定下来 

当javascript解释器调用一个函数时，它首先将scope chain设置为函数被定义时的scope chain。然后创建一个call object，并放置到scope chain的最前面。call object里包含了所有的临时变量和参数 

据我的理解，当一个function返回一个嵌套function，并赋值给一个变量，就形成了一个闭包。这个返回的function（即闭包），仍然访问到当时的变量

# chapter 9

结合使用new操作符和function()，会创建一个新的对象。完整的过程是这样的：首先new出一个object。然后将function.prototype的所有属性映射到新object上。（注意，不是复制，而是建立了一个映射关系）最后以新object为对象调用function 

要注意，object本身并没有prototype属性，有prototype属性的是function。但是当function作为构造函数使用时，其prototype的所有属性，都会被映射到object上。但object自身，是没有prototype属性的 

函数的prototype属性是在其定义时，自动创建和初始化的。初始值是只有一个属性的对象，这唯一的属性是constructor，它反向指向这个函数自身 

原型的属性并不是从prototype对象复制到新对象上，而是一种映射关系。这有2个很重要的意义，一是使用原型对象会减少内存的开销。二是即使在创建了对象以后再修改原型，修改的结果也会体现在先前创建的对象上 

读取对象属性的过程。比如读取o.p，首先会查找o对象上是否有p属性，如果没有的话，那么会查找o的prototype对象上是否有p属性。这里要注意，由于prototype自身是一个对象，所以上述的读取过程也是适用的，即会先在prototype对象上查找是否有p属性，然后到prototype的构造函数的prototype对象上查找。这个过程是递归的，直到查找到Object对象 

写对象属性的过程，则不存在上述的继承现象。比如设置o.p = 23;那么如果o对象有p属性，则将p属性的值设置为23。如果不存在p属性，则创建p属性，并设置为23，而不是再去递归查找prototype对象 

所有的对象都从它们的构造函数的prototype对象上继承属性。那么它们是如何继承到Object类的属性的呢？这是因为prototype对象自身也是一个对象，它是由Object()构造函数创建的，所以prototype对象自身也从Object.prototype对象上继承了一部分属性

当在Complex对象上查找属性时，首先在对象本身上查找。如果属性没有找到，就查找Complex.prototype对象。最后，如果仍然没有找到，则在Object.prototype对象上查找 

在需要的时候，用以下方式可以实现从任何对象继承，而不只是从Object类继承
```
PositionedRectangle.prototype = new Rectangle( );
delete PositionedRectangle.prototype.width;
delete PositionedRectangle.prototype.height;
PositionedRectangle.prototype.constructor = PositionedRectangle;
```

typeof null == "object"，而typeof undefined == "undefined" 

判断一个对象的类型，有typeof,instanceof,constructor等多种方法 

鸭子类型：如果一个对象拥有类X定义的所有属性，那么就可以将该对象看做是类X的一个实例，无论它是不是用X()构造函数创建的。

# chapter 10

如果要编写能在多个模块间共享的javascript代码，要遵守的最重要的规则就是避免定义全局变量。一旦定义了全局变量，就有这些全局变量被其他代码无意修改的风险。由此引发的BUG是很难定位的 

一个模块不该定义超过一个symbol到全局命名空间中。另外有2条建议：如果向全局命名空间中添加了symbol，其文档应该清楚地描述出这模块是什么。（比如jQuery）symbol的名称和引入symbol的.js文件之间，应该有清晰的对应关系。（包括目录名和文件名） 这章好像是插件开发相关的，暂时用不到，后面的部分就没继续看了

# chapter 13

在客户端javascript中，Document对象表示HTML文档，Window对象表示显示文档的浏览器窗口 

Window是客户端中的global object，非常重要。包括alert()等方法，document等属性，都是window对象的属性 

Window对象有2个属性指向自身，分别是window和self，用任何一个都可以获取window对象 

在一个window中声明的全局变量，不是另一个window中的全局变量，因为不同的window有不同的global object。不过，有途径让另一个window中的javascript代码获取到第一个window中的全局变量 

客户端javascript采用的是事件驱动的编程模型。当一个事件发生时，浏览器尝试调用合适的事件处理函数来响应这个事件。所以，为了编写动态交互的客户端javascript程序，需要定义合适的事件处理器，并注册到系统中，这样的话，浏览器就可以在合适的时间调用它们了 

业界提倡编写非侵入的javascript代码。应该做到，将javascript代码写在单独的js文件里，再引入html；事件处理函数用js代码注册，而不是直接写在html中；将js代码分模块进行组织；即使js代码不可用，页面的功能依然可用，等等 

将javascript代码嵌入html有多种方式，推荐的方式是
```
<script src="../../scripts/util.js"></script>
```

用src方式引入的外部js文件，效果就如同直接用script写入一样。所以在同一个页面中引入的多个js文件，它们是可以共享变量的，但是要注意变量冲突的问题 

当包含javscript代码的HTML文件被读入浏览器的时候，javascript代码即被执行 

javascript代码可以直接写在url中，但这种方式很不好，应该尽量避免 

出现在script中的javascript语句是按照它们出现的顺序执行的。当一个文件包含多个script，则这些脚本按照它们出现的顺序依次执行。javascript代码的执行，是html文档读取和解析的过程的一部分

当有多个onload事件处理函数注册，浏览器会调用所有的处理函数，但是调用的顺序则无从保证 

文档解析已经完成之后绝对不能调用document.write()方法。这样做的话，将创建一个新的document，并覆盖掉现有的document，用户甚至没有机会看到现有的document 

javascript是单线程的。因此两个事件处理函数绝对不会同时执行 

单线程也带来一些问题：它意味着javascript代码不能执行太长时间。如果javascript代码执行太久，会延迟document的载入，用户直到代码执行完成之间，都看不到页面。如果事件处理函数执行太久，则执行期间浏览器会停止响应，用户可能会认为网页已经崩溃了 

一般来说，不推荐在文档解析过程中，对文档内容进行操作。经验丰富的javascript程序员通常采用的做法，是在文档解析完成后，再对文档进行操作。否则可能会出现一些奇怪的问题 

javascript程序一个不能回避的问题，就是跨浏览器兼容性 

网页过多地依赖javascript代码，可能会带来可访问性的问题。比如有的用户使用的是移动设备，和声音阅读器之类的，要注意这种情况。当然，在国内这种情况似乎不多 

为了安全性的考虑，客户端javascript有意屏蔽了很多功能，比如删除文件，创建网络socket等，因为这些功能可以被恶意javascript代码用来做一些危险的事，损害用户的安全 

javascript一般遵循“同源策略”，来保证安全性，不同的浏览器对同源策略的实现也不一样

# chapter 14

客户端javascript提供了setTimeout()和setInterval()这2个方法来支持java中的TimerTask 

Location是地址对象，可以通过window.location访问到，其中包含protocol,href等属性 

通过给location赋值，可以使浏览器载入另一个地址，默认是相对路径 

location.replace(url)，这个方法可以载入指定的url，但是会替换掉历史浏览记录中的当前记录，而不是创建一条新记录，这造成浏览器的back按钮不可用 

History对象是历史浏览记录，可以通过window.history访问。不过出于安全和隐私的考虑，这个对象没有实现设计的初衷。尽管如此，该对象还是提供了back()和forward()方法， 效果和点击浏览的按钮是一样的 

Screen是屏幕对象，可以通过window.screen访问。这个对象提供了关于尺寸和颜色的一些信息 

Navigator是浏览器对象，可以通过window.navigator属性来访问。该对象提供了浏览器有关的信息，在判断浏览器类型的时候可以用到。要注意的是，navigator里提供的属性，不一定是可靠的 

window.open()方法会创建弹出窗口。出于用户体验的考虑，这个方法只能在响应用户操作的时候使用，如果不是响应用户操作，则会失败。（被浏览器拦截）此方法接受4个可选的参数，第一个是URL，如果为空，则打开新窗口；第二个参数是window的名称，如果该名称已经存在，则open方法返回该窗口的引用，而不是打开一个新窗口；第三个参数是新窗口的打开选项；第四个参数只有在第二个参数是已存在的window name时才有效，如果true，则在历史浏览记录里替换当前记录，如果是false，则在历史浏览记录里创建一条新记录（默认行为） 

window.open()方法的返回值是新打开的window的引用，opener属性反向指向打开它的窗口。如果一个窗口是用户打开的，而不是javascript代码打开的，那么这个属性的值是null

window.close()方法可以关闭窗口，但是只能关闭自己创建的窗口 

window.moveTo(),moveBy(),resizeTo(),resizeBy()方法可以移动窗口位置，或者改变窗口大小，不过出于安全考虑，浏览器对这些方法都做了一些限制 

浏览器提供3种方法进行屏幕交互，分别是alert(),confirm(),prompt() 

window.status属性是状态栏的提示文字，不过这个属性在IE可写，在FF下似乎是只读的。浏览器似乎没有提供其他属性或者方法，所以FF下状态栏应该是不可编辑的 

window.onerror属性可以绑定一个函数，如果这样做的话，当javascript执行过程中发生错误时，就会调用这个绑定的函数。该函数具有3个参数，第一个是错误信息，第二个是引发错误的javascript文件URL，第三个参数是引发错误的代码行数 

通过window的parent,top,frames属性，可以使frames互相访问 

每个window或者frame可以通过window或者self属性引用自身 

每个window都有frames属性，该属性是一个Window Object的数组。如果window没有包含任何frame，则frames[]为空，并且frames.length的值是0 

window有parent属性，指向包含它的window对象。比如说，window的第一个frame可以通过以下代码，得到它的同级下一个frame的引用
```
parent.frames[1]
```

如果嵌套的frame层级太多，可以用parent.parent的方式来获取，也可以用top属性来直接得到最外层的window的引用 

对于最外层的window来说，window == self == parent == top。（也就是说parent.parent.parent...和top.top.top...代码是合法的，而且是无限的，都指向自身）

可以给window和frame指定name属性，这样做的目的是在a和form标签中，可以指定浏览器在哪个窗口显示链接，或者提交表单
```
<a href="chapter01.html" target="mainwin">Chapter 1, Introduction</a>
```

给frame指定了name属性以后，会在window中创建同名的属性，这个属性指向该frame
```
parent.table_of_contents
parent.frames[1]
```
可以看到，这样获取frame对象，比用数组index来得方便，也更加直观 

每个window和frame都有各自独立的execution context，也就是scope chain。所以每个window和frame都有各自的global object，即window对象。但是通过parent.frames[0].i，frame可以读取到另一个frame中定义的变量或者方法。但是需要注意的是，由于function是文法作用域，而不是动态作用域，所以当在frameB中调用frameA中定义的函数f时，f是在frameA的scope chain上查找属性，而不是在frameB的scope chain上查找

# chapter 15

document.write()方法只能在html文档解析过程中调用，如果在解析完成之后才调用，则会创建一个新的文档，并将旧文档覆盖。因此，有时候可以用如下代码创建新窗口
```
function hello() {
    var w = window.open();             //    Create a new window with no content
    var d = w.document;                // Get its Document object
    d.open();                             // Start a new document (optional)
    d.write("<h1>Hello world!</h1>");  // Output document content
    d.close();                           // End the document
}
```
不过这种做法现在已经比较少见了 

document有一个referrer属性，该属性包含了用户链接到当前文档的文档的URL。（也就是跳转前的url） 

document对象有一些数组属性，可以作为访问html元素的快捷方式，包括包括forms等，不过由于是采用下标方式访问，比如forms[0]代表第一个form元素，所以对文档结构的稳定性有较高要求 

这些属性（forms[]等）是可以编程的，但是要注意不能改变文档的结构，因为这些遗留的DOM API不允许在reflow的情况下改变文档 

为了更方便地访问到文档元素，可以给元素命名
```
<form name="f1"><input type="button" value="Push Me"></form>
document.forms[0]     // Refer to the form by position within the document
document.forms.f1     // Refer to the form by name as a property
document.forms["f1"]  // Refer to the form by name as an array index
```

事实上，给form, img, applet（注意，不包括a）这些元素命名，相当于在document里定义了同名属性，就可以直接访问了，而不需要通过对应的数组元素，如 document.f1 

form中的子元素也可以命名，然后直接访问到
```
<form name="f1">
    <input type="text" name="t1" />
</form>
```

然后就可以用document.f1.t1来找到input元素。而且要注意，这个name属性同时也是提交表单时提交到后台的param名，如果和struts2共同作用，就会绑定到action的t1 field上 

绑定事件处理函数，在html和javascript中的写法不同
```
<form name="myform" onsubmit="return validateform();">...</form>
```

```
document.myform.onsubmit = validateform;
```

在javascript中，只需要给form的onsubmit property绑定一个函数。而在html中，需要在onsubmit attribute上实际调用这个函数

W3C DOM标准扩展并取代了遗留的DOM标准 

W3C DOM把文档看做树形结构 

每个Node Object都有nodeType属性，该属性表示该Node的节点类型。如果其nodeType的值等于Node.ELEMENT_NODE，就表示该节点是一个元素节点，也就是说它是一个Element Object，那么就可以使用Element接口定义的所有属性和方法 

DOM树的根节点是Document Object，其nodeType值是Node.DOCUMENT_NODE，也就是9。它的documentElement属性，指向该文档的根元素。对HTML文档来说，也就是html标签。总结来说，DOM树首先是Document Object，可以用document得到，然后document的根元素是html

用document.documentElement属性和document.childNodes\[1\]属性，都可以取到html，它是一个HtmlElement。（document.childNodes\[0\]是DocumentType） 

用document.documentElement.childNodes\[1\]和document.body，都可以取到body，它是一个BodyElement。（document.documentElement.childNodes\[0\]是HeadElement） 

ElementNode的nodeType值是1，AttributeNode的nodeType值是2，TextNode的nodeType值是3，这几个是比较常见的 

在HtmlElement上使用childNodes[]属性来获取子元素这个方法，应该尽量避免使用，因为这个方法在跨浏览器上有很大的问题。比如对于以下页面想获取到img元素
```
<body>
    <button id="b1">click me</button>
    <img src="abc" name="i1" />
</body>
```
在FF下是document.body.childNodes\[3\]，但是在IE下是document.body.childNodes\[2\]。发生这个现象的原因是FF和IE对空白的处理不同，FF会把空白看做是TextElement，而IE会忽略这些空白。因此，用childNodes[]来获取子元素，是很危险的做法 

在ElementNode上使用getAttribute()方法，可以获取元素的特性值(attribute)。也可以用getAttributeNode()方法，获取到AttributeNode。不过这个方法很不方便，一般就是直接使用getAttribute()方法 

DOM标准也包括了特别为HTML文档制定的接口。比如HTMLDocument是Document的子接口，HTMLElement是Element的子接口。（好比HttpServletRequest是ServletRequest的子接口一样）此外，DOM还为很多HTML元素定义了标签特定的接口。比如HTMLBodyElement，HTMLTitleElement等，它们大部分定义了对应HTML标签attribute的属性集合(properties)

HTMLElement元素定义了id, style, title, lang, dir, className属性。这些属性对应到html标签的id, style, title, lang, dir, class这些attribute（特性）。这些特性是所有HTML标签都支持的 

之所以为HTML标签定义相应的HTMLElement接口，主要是为了方便。这些接口通常只是增加一组对应HTML attributes的properties。比如要获取一个img标签的src attribute，要使用img.getAttribute("src")方法，比较麻烦。有了对应的专用HTMLElement接口以后，就可以直接使用img.src，来获取src attribute，方便了不少 

综合来说，要获取
```
<img name="i1" src="abc.jpg" />
```

这个标签中的src attribute的值，有很多种方法
```
document.documentElement.childNodes[1].childNodes[3].getAttribute("src");
document.body.childNodes[3].getAttribute("src");
document.body.childNodes[3].src;
document.images[0].src;
document.i1.src;
```
只有熟悉了DOM的API（包括遗留API和W3C标准API），才能比较快速正确地获取想要的元素和值，当然，现在最方便的方法是用jQuery 

DOM有1,2,3三个级别，从DOM 2开始，DOM划分了模块，并且引入了CSS和EVENT的支持。DOM3的特性不太清楚。还有史前的DOM 0，也就是遗留DOM 

hasFeature()这个特性检测方法其实意义不大，因为这个方法也是浏览器厂商自己实现的，是否支持特性由浏览器厂商自己说了算。所以做特性检测时，不应该依赖这个方法 

IE6里不支持NODE接口的NODE_TYPE常量，所以如果要用的话，需要自行定义 

尽管DOM标准是来自于对动态HTML编程通用API的需求，但是DOM不只关注网络脚本。实际上，该标准现在已经广泛使用在服务器端编程，用于解析和操作XML文档。由于其广泛使用，现在DOM标准已经定义为语言独立的标准 

对于DOM规范中定义的方法，在不同的语言中可以有不同的实现。比如DOM规范中定义，实现应该提供“获取第一个子节点”的能力。在java中，实现是getFirstChild()方法，而在javascript中，则是一个firstChild属性。这两种实现不同，但是都实现了DOM规范

Node的childNodes属性的值是一个NodeList Object，该对象的行为类似于Node Object的数组 

操作DOM时有一个基本的规则：在文档树加载解析完成之前，不能对其进行遍历或者操作。SAX不是这样的 

Node接口除了定义了childNodes属性外，还定义了其他几个方便的属性，包括firstChild, lastChild, nextSibling, previousSibling 

每个DOM tree的根节点都是Document Object（文档对象，在客户端javascript里就是window.document），但它并不代表树里的某一个HTML元素。document.documentElement属性指向<html>标签，它作为文档的根元素 

document.getElementsByTagName()方法，根据tag名称获取ElementNode的数组集合，然后用下标访问
```
var tables = document.getElementsByTagName("table");
alert("This document contains " + tables.length + " tables");
```

注意，getElementsByTagName的返回结果是不区分大小写的，传递的参数也是 

document.getElementById()方法，是根据id attribute获取Element，只会返回一个结果 

除了Document Object之外Element Object也有getElementsByTagName方法，用法和document的同名方法一样，区别在于它只搜索调用元素的子元素，不会遍历整个DOM tree 

document.getElementsByName()方法是根据name attribute查询，返回结果是一个数组 

可以用setAttribute()方法设置元素的attribute，也可以直接赋值的方式来设置，2者的效果是一样的 

DocumentFragment是一种特殊的节点类型，它本身不出现在文档中，而是作为节点集合的临时容器存在，并且能够把这些节点集合看做一个单独的对象进行操作。当把一个DocumentFragment插入文档时，并不是其自身被插入，而是它所有的子节点被插入，之后DocumentFragment就被清空，并且不能再次使用，除非重新向其中加入子节点

document.createElement()和document.createTextNode()方法可以创建新的元素，以及新的文本 

Node.appendChild(),Node.insertBefore(),Node.replaceChild()方法，可以将新创建的元素，加入到文档中 

总结前2条，就是创建新节点（包括元素节点和文本节点），都是通过调用document上的create方法完成的。但是要将新创建的节点显示到浏览器上，还需要调用Node上的相关方法 

HTMLElement Object还定义了一个innerHTML属性，这个属性不是W3C标准，不过由于比较方便，使用范围很广，而且现代浏览器都支持。该属性的值，是一段表示所有子元素的HTML文本。如果对这个属性赋值，浏览器会调用其HTML解析器，解析传递参数的值，并用解析的结果替换掉原本的子元素

非常早的IE（IE4，或许还有5和6），提供了一套非标准的API，用于实现遍历文档的功能。比如document.children，功能类似于W3C的document.childNodes。这些API现在已经基本用不到了 

早期的IE还提供了另外一些乱七八糟的API，比如document.all什么的， 现在也是基本看不到了，就不多说了

# chapter 16

浏览器内置有默认的样式表，用户可以用自己的设置来改变默认值。这也是为什么页面在不同的浏览器中视觉效果不同的原因 

通过一个ElementNode的style属性，可以编辑该元素的一些CSS属性。不同的浏览器对该属性的实现是不同的：
```
var s = document.i1.style;
var count = 0;
for ( var prop in s) {
    count++;
}
alert(count);
```

执行上面的代码，FF下支持202个属性，IE下只有148个，Google浏览器只有11个 

element.style属性获取的CSS2Properties对象，只包含内联的css样式，通过外部css文件设置的属性，用style属性无法取到。同样，设置这个值，实际上设置的是内联样式，将覆盖外部css文件的设置值
```
var p1 = document.getElementById("p1");
var s = p1.style;
alert(s.color);
```

```
<body>
	<button id="b1">click me</button>
	<p id="p1" style="color:red;">hello world</p>
</body>
```

将显示"red"，如果是用外部css文件设置，则没有值 

CSS属性是多个单词用-连接的，在javascript里会改用驼峰方式命名
```
element.style.fontFamily = "sans-serif";
```

给style的属性赋值时，一定是使用string 

所有的位置属性都需要单位
```
element.style.left = "300px";
```
同样，取出位置属性，也是带有单位的string 

window对象的getComputedStyle()方法，返回CSS2Properties对象。不同于element.style属性只包含内联样式，这个返回值包含了内联样式和外部CSS文件定义的样式。该方法接受两个参数，第一个参数是要计算的元素，第二个参数是CSS的伪类。FF和Google浏览器支持这个方法
```
var p1 = document.getElementById("p1");
var s = getComputedStyle(p1, null);
alert(s);
for ( var prop in s) {
    alert(prop + ": " + s[prop]);
}
```

IE不支持上述方法，但是它提供了类似的功能。每个元素都有currentStyle属性 

可以设置外部CSS文件是否生效
```
<link id="css1" type="text/css" rel="stylesheet" href="../styles/css.css" />
```

```
function toggleCssSheet() {
    var c1 = document.getElementById("css1");
    c1.disabled = !c1.disabled;
}
```

FF还支持直接操作CSS样式表文件，不过作用不大
