title: Simple Javascript Template
date: 2013/07/29
---

>经常碰到拼接字符串时候，平时都是用[泽飞](http://www.planabc.net "泽飞")写的substitute函数，但老是忘记那个函数，现给它转截给过来，先记着

转载地址：http://www.planabc.net/2011/05/31/simple_javascript_template_substitute/

我们在平常使用字符串拼接的时候（如下例），会发现代码的可维护性和易读性将变得更加糟糕（代码中一堆的变量、双引号、单引号, 加号等，相信当情况更为复杂时，头一定发晕）:


``` js
  var url= 'http://www.plannabc.net/',
  title= '落草为根——专注前端技术&关注用户体验',
  text = '怿飞's Blog';

	var link = '<a href="' + url + '" title="' + title+ '">' + text+ '</a>';

```

如果上述代码变为：

``` js
  var obj = {
    url: "http://www.plannabc.net/",
    title: "落草为根——专注前端技术&关注用户体验",
    text: "怿飞's Blog"
	};
	var link = '<a href="{url}" title="{title}">{text}</a>';
	substitute(link, obj)
```
一切变得怡然自得。

substitute 函数的实现思路其实很简单：使用 String 的 replace 函数，在 replace 函数中用正则匹配除模板中的要替换的标签（“{key}”），并进行替换：


``` js
  function substitute (str, obj) {
    if (!(Object.prototype.toString.call(str) === '[object String]')) {
        return '';
    }

    // {}, new Object(), new Class()
    // Object.prototype.toString.call(node=document.getElementById("xx")) : ie678 == '[object Object]', other =='[object HTMLElement]'
    // 'isPrototypeOf' in node : ie678 === false , other === true
    if(!(Object.prototype.toString.call(obj) === '[object Object]' && 'isPrototypeOf' in obj)) {
        return str;
    }

    // https://developer.mozilla.org/en/JavaScript/Reference/Global_Objects/String/replace
    return str.replace(/\{([^{}]+)\}/g, function(match, key) {
        var value = obj[key];
        return ( value !== undefined) ? ''+value :'';
    });
	}
```

substitute 函数将模板中的标签 {key} 替换成 obj 中对应的 value（obj[key] or obj[key].toString()），如果不存在，则替换成空字符。

如果模板中某些内容不需要替换的怎么办？比如：{some text need brace}



1. 可以增加新的语法，人为控制不需要替换的模板标签，比如：\{key\}

2. 使用更为少见的字符作为模板标签，避免与常规情况撞车，比如：{{key}}
当然如果输入的数据 obj 为不完全信任的数据（比如：XSS）时，可以增加字符的转义：


``` js
  function escape(s) {
    s = String(s === null ? "" : s);
    return s.replace(/&(?!\w+;)|["'<>\\]/g, function(s) {
      switch(s) {
        case "&": return "&amp;";
        case "\\": return "\\\\";
        case '"': return '&quot;';
        case "'": return '&#39;';
        case "<": return "&lt;";
        case ">": return "&gt;";
        default: return s;
      }
    });
	}
```
我们再看看 YUI3 中的实现 ：https://github.com/yui/yui3/blob/master/src/substitute/js/substitute.js
YUI3中做了更多的处理 substitute(s, o, f, recurse)：

1. 允许传入 fn，fn 将对 obj 中对应的 key/value 进行处理，返回新的 value。
2. 如果支持 Y.dump 函数，将把 value 是对象的转成一定格式的字符串，如果不支持，直接返回对象的 toString。
3. value 值为非对象、非字符串和非数字时 ，保持原标签不作替换。
4. 如果 recurse 参数设置为 true，将进行标签的递归替换。
其实一般的时候简单的方式就够用了。面对自己编写库或组件的时候，都会有如下的选择：
1. 大而全，涵盖所有的扩展，能捕捉到所有的异常。
2. 小而精，满足一般的需求，特殊情况可定制，异常可通过约定控制。
选择最合适的才是最重要的，对于自己，通常偏向后者。

