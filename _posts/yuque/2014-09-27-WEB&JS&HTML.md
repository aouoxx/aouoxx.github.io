---
layout: post
title: web基础知识
categories: [web, js, html]
description: web基础知识的介绍,js和html的相关信息
keywords: web, js, html
---

<meta name="referrer" content="no-referrer"/>
​

​

​

​

#### js 中 date 的介绍

```java
获取当前时间
	var now = new Date();
获取当前时间后一天或后面一个小时的时间
    var now  = new Date();
    now = now.setDate( now.getDate( )+1 );
    hour = now.setHour(now.getHours( )+1);


获取当前时间戳（以 s 为单位）
    var timestamp = Date.parse(new Date())
    console.log(timestamp);
    1442825058000
获取某个格式的时间戳
	var stringTime =

```

​

```java
/日期时间原型增加格式化方法
Date.prototype.Format = function (formatStr) {
    var str = formatStr;
    var Week = ['日', '一', '二', '三', '四', '五', '六'];
    str = str.replace(/yyyy|YYYY/, this.getFullYear());
    str = str.replace(/yy|YY/, (this.getYear() % 100) > 9 ? (this.getYear() % 100).toString() : '0' + (this.getYear() % 100));
    var month = this.getMonth() + 1;
    str = str.replace(/MM/, month > 9 ? month.toString() : '0' + month);
    str = str.replace(/M/g, month);
    str = str.replace(/w|W/g, Week[this.getDay()]);
    str = str.replace(/dd|DD/, this.getDate() > 9 ? this.getDate().toString() : '0' + this.getDate());
    str = str.replace(/d|D/g, this.getDate());
    str = str.replace(/hh|HH/, this.getHours() > 9 ? this.getHours().toString() : '0' + this.getHours());
    str = str.replace(/h|H/g, this.getHours());
    str = str.replace(/mm/, this.getMinutes() > 9 ? this.getMinutes().toString() : '0' + this.getMinutes());
    str = str.replace(/m/g, this.getMinutes());
    str = str.replace(/ss|SS/, this.getSeconds() > 9 ? this.getSeconds().toString() : '0' + this.getSeconds());
    str = str.replace(/s|S/g, this.getSeconds());
    return str;
}

调用的时候设置的方法：
  var d = new Date();
  var str = d.format("yyyy-MM-dd hh:mm:ss");
  console.log( str )

```

​

#### window 的使用

```java
页面延时跳转
	页面延时2s后跳转
	window.setTimeout("window.location.href='xxxx'",2000);

```

​

​

​

### JQuery 选择器

```html
#id 选择器 JQuery
能使用CSS选择器来操作网页中的标签元素。如果想要通过一个id号去查找另一个元素就可以使用下面格式的选择
$('#my_id')
其中my_id表示根据id选择器获取页面中的指定的标签元素，且返回一个唯一的元素。
element元素选择器 $("element")就是元素名称
<body>
  <div>1</div>
  <div>2</div>
</body>
获取所有div元素的选择器：（修改所有div元素字体的样式）
<script type="text/javascript">
  $("div").css("font-weight", "bold");
</script>

.class选择器 $(".class") 其中.class参数表示元素的css类别(类选择器)名称。
<div class="red">my red</div>

<script type="text/javascript">
  var $className = $(".red").attr("class");
</script>

$(“*”) 获取全部的元素 选择器中的参数就一个“*”， 既没有“#” 号，也没有 “.”号。
由于该选择器的特殊性，它常与其他元素组合使用，表示获取其他元素中的全部子元素。
sele1 sele2 seleN 选择器 调用的格式如下： $("sele1 , sele2 ,seleN") 参数sele1
sele2到seleN为有效选择器,每个选择器之间用","号隔开，它们可以是前天提到的各种类型的选择器
如 $('#id') $
```

​

### JQuery 事件

```html
ready()方法 $(doxument).ready(fucntion(){ }) $().ready(function(){ })
$(function(){ }) 当文档加载后激活函数： $(document).ready(function()){ }
由于该事件在文档就绪后发生，因此把所以其他的jquery事件和函数置于该事件中是非常好的做法。
ready（）函数规定当ready事件发生时执行的代码
ready（）函数仅能用于当前文档，因此无需选择器 Note: ready() 函数不能和
<body onload="">
  一起使用
</body>
```

```html
bind（）方法
bind方法为被选元素添加一个或多个事件处理程序，并规定事件发生运行的函数
将事件和函数绑定到元素：规定向被选元素添加一个或多个处理程序，以及当事件发生时运行的函数
$(selector).bind(event , data, function) event:
规定添加到元素的一个或多个时间，由空格分隔成多个事件 data:
规定传递到函数的额外数据 function: 规定当事件发生时运行的函数
$(selector).bind({event:function,event function,...}) 当点击鼠标时，隐藏或显示P
元素 $('botton').bind('click',function(){ $('p').slideToggle(); })
```

```html
one（）方法
one（）方法为被选元素附加一个或多个事件处理程序，并规定事件发生时运行的函数
当使用one（）方法时，每个元素只能运行一次事件处理器函数 $(selector).one( evnet ,
data, function ) event: 规定添加到元素的一个或多个时间，由空格分隔成多个事件
data: 规定传递到函数的额外数据 function: 规定当事件发生时运行的函数
```

```html
trigger（）方法 trigger（）方法触发被选元素的指定事件类型
$(selector).trigger(event,[param1,param2]) event
规定指定元素要触发的事件。可以使自定义事件(使用bind（）函数来附加)，或者任何标准事件
[param1,param2] 传递到事件处理程序的额外参数。额外的参数对自定义事件特别有用
```

​

### JQuery 处理事件

```html
获取焦点事件 鼠标事件 mousedown(function): 点击鼠标后触发事件
mousemove(function); 鼠标在元素上移动触发事件 mouseout(function)
鼠标从元素上离开后触发 mouseover(function) 鼠标移入对象时触发 mouseup(function)
鼠标点击释放后触发
```

​

​

#### 设置 checkbox

```javascript
获取单个checkbox选中的写法:
  $('input:checkbox:checked').val();
  $("input:[type='checkbox']:checked").val();
  $('input:[name='ck']:checked').val();

获取多个checkbox的选中项
  $('input:checkbox').each(function(){
       if($(this).attr('checked')==true){
       alert($(this).val());
      }
  })

设置第一个checkbox为选中值
  $('input:checkbox:first').attr('checked','checked');
   或者
  $('input:checkbox').eq(0).attr("checked",true);

根据索引设置任意一个checkbox为选中值
  $('input: checkbox').eq(索引值).attr('checked',true);
  或者
  $('inut:radio').slice(1,2).attr('checked,true');

根据Value的值设置checkbox为选中值
	$("input:checkbox[value='ss']").attr('check','true');

删除Value为1的checkbox
	$("input:checkbox[value='ss']").remove();

删除第几个checkbox
	$('input:checkbox').eq(索引值).remove() 索引值=0,1,2，，，

全选Checkbox
  $('input:checkbox').each(function() {
          $(this).attr('checked', true);
  });

```

​

#### select 选中值

```javascript
<select class="selector"></select>

1) 设置选中值为pxx的选项
 		$('.selector').val('pxx');
2) 设置text 为pxx的选项
		$('.selector').find("option[text='pxx']").attr('selected',true)
3) 获取当前选中项的value
	  $('.selector').val();
4) 获取当前选中项的text的值
 		$('.selector').find('option:selected').text();
```

​

​

​
