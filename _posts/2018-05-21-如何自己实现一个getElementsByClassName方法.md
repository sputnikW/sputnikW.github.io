---
layout: posts
category: front_end
---

`document.getElementsByClassName()`只有在IE9+的浏览器中才可以使用，所以如果我们想要在之前的IE浏览器版本中使用这个功能，就需要自己去实现，下面就来讨论一下实现过程。
***
### 基本思路：

1. 获取页面中的所有DOM元素
2. 遍历所有元素，检查其class属性的值
3. 通过正则表达式匹配，判断该元素是否含有该类名，如果满足则存入结果数组中。

***

### 需要考虑的问题：

1. 原生`document.getElementsByClassName()`方法获取的是一个`HTMLCollection`，也就是一个**动态集合对象**，意味着如果通过该方法查询的结果存在一个变量中，当DOM中的节点有所变化的时候，变量中的内容也会动态的更新。因此理论上我们应该获取`HTMLCollection`类型的结果，但因为`HTMLCollection`对象的构造函数比较复杂，所以考虑到实际使用情况，我这里就只返回一个数组。
2. 获取DOM中所有元素的方法，通常情况下，可以获取所有元素方法有以下几种：
	- `querySelectorAll()`,但是这个方法的兼容性也为IE9+(IE8仅支持CSS2选择器)，
	- 递归遍历`document.childNodes`属性,此属性会返回DOM中的文本节点，比如两个标签之间的换行符，代码缩进等，所以也不合适。
	- `getElementsByTagName("*")`方法，此方法兼容到IE6+，所以足够用了，这也是我在本文中使用的方法。

3. 正则表达式的匹配，因为class属性是一个字符串，而只要这个字符串中含有我们要查询的className就满足匹配的条件，所以有几下几种情况：
	- 字符串中该类名两边都有空格（或其他空白符，如`\t`等），例如:	`class="bob alex tom"`中，`alex`的两边都有空格，说明存在`alex`这个类名，而不会匹配`class="bob alexaa tom`（右边紧跟的不是空格）
	- 如果类名的某一遍没有空白符，则必须是字符串的开头或者结尾。例如`class="bob alex tom"`中的`bob`也应该是匹配成功的。
	- 只需要字符串中有一个匹配成功，就算是匹配成功，比如类名重复的状况。
4. ***需要注意的是，我们创建的方法返回的是一个节点数组，而不是`HTMLCollection`或`NodeList`，所以不具有动态更新的特点。***
5. ***另外需要注意的是，原生`document.getElementsByClassName()`方法的参数是类字符串，例如`document.getElementsByClassName(" john   bob  ")`获取的是既有`john`类又有`bob`类的元素，一次可以查询多个类名，在本文中我实现的方法则一次只能查询一种类名。***  如果想要查询所有类名，应该对我们自定义函数的参数进行处理，获取真实的类名集合，然后与节点类名字符串进行匹配。
***  
下面开始代码实现：

	function MyClassSelector(className){
		className = className.trim()
		var result = [] 
		
		var nodelist = document.getElementsByTagName("*")   
		// 注意：nodelist此时是一个HTMLCollectionl,这是一个对象（可以用typeof测试一下看看），
		// 除了数字索引和DOM节点元素组成的键值对外，还有其他属性，如length等
		
		var re_string = "(\\s|^)"+className+"(\\s|$)"  // 注意斜杠需要转义
		var re = new RegExp(re_string)  // 生成正则表达式对象 /(\s|^)className(\s|$)/
		
		for(let i=0; i<nodelist.length; i++){
			let className = nodelist[i].className // 获取节点的className
			if(re.test(className)){
				result.push(nodelist[i]) 
			}
		}
		return result
	}
	

下面是提供测试的HTML：

	<body>
	<div class="john	bob">
	  <div>
	    <p>
		  <span class="mike john bob"></span>
		</p>
	  </div>
	</div>
	<div class="john">
	  <p class="john2">
	  </p>
	</div>
	</body>

测试结果：

	>>> a = MyClassSelector("john")  // 控制台输入
	>>> Array [ div.john.bob, span.mike.john.bob, div.john ]  //输出
	

