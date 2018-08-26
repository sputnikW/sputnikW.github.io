### 概述

JavaScript中的面向对象是基于原型链来实现的，这不同于其他语言复制拷贝的方式。我觉得原型链的好处是节约内存，提高性能，缺点可能就是不那么容易理解。

下面我们就来循序渐进的通过原型链，来理解JavaScript中的面向对象。

### 面向对象的概念是为了解决什么问题?

如果我们想创建一个具有一定功能的集合，在JavaScript中我们可以这样写：

```javascript
var Animal = {
    name: 'kitty',
    sleep: function(){
        console.log(this.name + " is sleeping");
    }
};
```

即通过对象的形式，将一些属性（如`name`）或方法（如`sleep`）包装在一起，这样就形成了逻辑上的集合。

但是如果我们想再创建一个类似的对象呢？最直接的方式是这样：

```javascript
var Animal = {
    name: 'kate',
    sleep: function(){
        console.log(this.name + " is sleeping");
    }
};
```

也就是重新再写一个对象，修改其中的部分属性。这种方式无疑是非常僵硬的，因此可以通过这种“工厂函数“的方法：

```javascript
function createAnimal(name){
    var obj = {}
    obj.name = name;
    obj.sleep = function(){
        console.log(this.name + " is sleeping");
    }
    return obj;
}
```

可以看到，”工厂函数“的作用，即在内部创建一个空的对象，然后将属性和方法填充进去，最后再返回这个对象。

这样看起来似乎没什么问题，但是仔细想想，其实我们只不过是想按照一定的模式创建一个对象，逻辑上来讲，我们要做的应该只是提供模式，而新建一个空对象，返回一个空对象的操作其实没有必要由我们来完成，因此JavaScript就提出了一个`new`关键字。这个关键字的用法很简单：

```javascript
function Animal(name){
    this.name = name;
    this.sleep = function(){
        console.log(this.name + " is sleeping");
    }
}

var kitty = new Animal("kitty");

//如果我们在浏览器控制台打印kitty，得到的是这样的一个对象：
{
 name: "kitty"
 sleep: function sleep()
}
```

即在调用Animal函数前，加上一个new关键字，告诉编译器，这个函数是一个”构造函数“，我们应该调用这个函数，并且根据函数所指定的规则，包装一个对象，并返回这个对象。

***tips：后面在了解的原型链后我们会自己实现一个new***

### JavaScript中的原型链

前面我们已经知道了为什么需要面向对象，下面我们就来看看JavaScript语言处理面向对象的巧妙之处——原型链

先来看一张图

![原型链](https://github.com/sputnikW/web-notes/blob/master/images/prototye-list.jpg)

先来从上往下讲解一下这张图：

- **函数**（`function`）都有一个`prototype`属性，指向一个**原型对象**。
- **原型对象**都有一个`constructor`属性,指向这个原型对象对应的构造函数
- **普通对象**都有一个`__protor__`(ES并没有规定这个属性的标准名称，而浏览器大多都用`__proto__`访问这个属性)，指向这个**普通对象**的**原型对象**。

我们还是拿上面的那个例子来看看：

```javascript
function Animal(name){
    this.name = name;
    this.sleep = function(){
        console.log(this.name + " is sleeping");
    }
}
var kitty = new Animal("kitty");
```

我们先在浏览器中打印出kitty：

![kitty对象](https://github.com/sputnikW/web-notes/blob/master/images/Object.jpg)

可以看到kitty即一个对象，这个对象直接包含name属性和sleep方法。可以看到还有一个灰色的属性：`<prototype>`对象，这就是前面我们提到的`__proto__`，（完全是同一个东西，只不过火狐浏览器这样显示罢了，为了保持统一，我们在本文就叫它`__proto__`）。

我们展开这个`__proto__`看看它的组成：

![__proto__](https://github.com/sputnikW/web-notes/blob/master/images/__proto__.jpg)

很简单嘛，也就是一个包含constructor属性的对象，这个constructor属性指向kitty的构造函数Animal。

注意，在这里，`__proto__`所指向的对象也有一个`<prototype>`，这是理所当然的，因为这个属性所有的对象都有（甚至函数也有，因为函数其实也是一种对象）



我们再在浏览器中打印出`Animal`这个函数：

![构造函数](https://github.com/sputnikW/web-notes/blob/master/images/constructor.jpg)

可以看到这个函数有一个prototype属性,我们已经可以看到这个prototype属性是一个对象（即**原型对象**）。我们再展开这个prototype属性看看：

![prototype](https://github.com/sputnikW/web-notes/blob/master/images/prototype%E5%B1%9E%E6%80%A7.jpg)

可以看到目前这个原型对象非常简单，就是一个constructor，指向了Animal这个函数。

那么如果我们想原型对象中添加一些东西呢？像这样：

```javascript
Animal.prototype.eat = function (food){
    console.log("eat " + food);
}
```

加上这段代码，我们刷新一下页面，再看看这时的原型对象是什么：

![eat](https://github.com/sputnikW/web-notes/blob/master/images/prototype_eat.jpg)

果然，eat 方法被添加到了Animal的原型对象中。我们在kitty中调用这个方法试试看：

```javascript
kitty.eat("shxt");
//打印结果：eat shxt
```



好，到此为止，我们已经大致了解了原型链的存在形式：即通过原型对象来链接，下面我们总结一下：

- 创建一个函数的时候，会创建这个函数对应的原型对象，原型对象的`constructor`指向这个函数。
- 函数的`prototype`属性，以及函数（通过`new`）创建的实例对象的`__proto__`属性，都指向同一个原型对象。

而在原型链的作用在于：

比如上面我们在Animal的原型对象上定义了一个eat方法。我们有一个Animal的实例kitty，我们打印出kitty：

![kitty对象](https://github.com/sputnikW/web-notes/blob/master/images/kitty.jpg)

发现kitty对象中并没有eat方法，那么它是怎么调用到eat的呢？没错，我们可以看到kitty对象的原型对象中有eat这个方法。所以在原型链机制中，对象的方法和属性的调用过程是这样的：

1. 先在自己当前对象寻找这个属性或方法，如果找到了就直接用
2. 如果找不到，就去当前对象的`__proto__`属性指向的原型对象中寻找，如果找到了，就使用
3. 如果还找不到，就在当前原型对象的`__proto__`属性指向的上一级原型对象寻找
4. 如果没有更上一级的原型对象，那么`__proto__`属性会指向`Object`的原型对象（这个对象就没有`__proto__`属性了）
5. 如果在Object的原型对象中还找不到，那么就返回`undefined`

`Object`的原型对象长这样：

![Object的原型对象](https://github.com/sputnikW/web-notes/blob/master/images/Object's_prototype.jpg)

是不是看着很熟悉，里面有很多我们常用的方法。

那就顺便再把function的原型对象放出来：

![function的原型对象](https://github.com/sputnikW/web-notes/blob/master/images/function_prototype.jpg)

是不是更熟悉了？而且可以看到function的原型对象的`__proto__`属性，指向Object的原型对象。



所谓原型链，就是一个对象的`__proto__`指向一个原型对象，而这个原型对象的`__proto__`又指向另一个原型对象，依次串联下去。

目前我们谈论的原型链都是很短的，要想凸显原型链的威力，就要讲讲继承了。

### JavaScript中的继承

所谓继承就是，在一个父类的基础上，创建一个子类，子类拥有父类的属性和方法，也有自己的属性和方法。

在JavaScript中，实现继承主要思想是：

拿到子类的构造函数的原型对象，将其`__proto__`属性指向父类构造函数的原型对象。

这样子类构造函数生成的实例对象就可以先访问到子类的原型对象，再访问到父类的原型对象，就完成了继承。

具体的实现方式呢，有多种：

##### 一、实现子类构造函数，在子类构造函数上修改原型链

这种是最传统的做法，步骤是：

- 创建一个子类构造函数
- 在这个构造函数内调用父类的构造函数
- 将子类构造函数的原型对象的`__proto__`属性指向父类构造函数的原型对象

这样，当子类创建的实例中寻找属性或方法时，先找到子类的原型对象，找不到的话就去子类原型对象的`__proto__`（即父类的原型对象）找，这样就实现了继承。

我们来实现一个例子：

```javascript
function Animal(name) {
    this.name = name;
    this.sleep = function () {
        console.log(this.name + " is sleeping");
    }
}
Animal.prototype.eat = function (food) {
    console.log(this.name + " is eating " + food);
}

function Cat(name, color) {
    Animal.call(this, name); //获得Animal内部定义的属性和方法
    this.color = color;
}
//Object.create()方法作用是，创建一个空的对象，将这个对象的__proto__属性指向参数
Cat.prototype = Object.create(Animal.prototype); //获得Animal原型对象上的方法和属性
//这里暂时先不考虑原型对象的constructor属性的指向是否正确
//我们打印出来看看Cat构造函数
console.log(Cat);

//下面我们可以新建一个Cat实例
var kitty = new Cat("kitty","yellow");
console.log(kitty);
//试试使用父类的原型上的方法
kitty.eat("fish"); //console: kitty is eating fish
```

你可以复制我的代码到浏览器的控制台，看看打印出来的结果是怎样的。

但是这种方法有一个缺点，即对于某些JavaScript内建对象（如Date），如果实例对象不是由它本身的构造函数生成的，不能访问其内部的属性和方法。所以通过这种方法继承Date类，即便我们修改了原型链，但还是不能调用Date内的方法。不信？我们来验证一下：

```javascript
function MyDate(date) {
    Date.call(this, date);
    this.log = function () {
        console.log("now is " + date);
    }
}
MyDate.prototype = Object.create(Date.prototype);

var time = new MyDate("2018-08-23");
time.getDate(); //console: TypeError: getDate method called on incompatible Object
```

虽然我们已经将MyDate的原型对象的属性指向Date的原型对象了，但是还是不能调用Date中的方法。

##### 二、创建父类实例，在父类实例上修改原型链

解决上面的问题其实也很简单，即我们先用Date构造函数生成一个实例对象，然后将这个实例对象改造成子类实例对象，这样就不会出现上面这种问题。所以这种方式的步骤是这样的：

- 用父类构造函数创建实例
- 将实例的`__proto__`属性指向子类的原型对象
- 将子类原型对象的`__proto__`指向父类的原型对象

这样，父类原型对象又被链接到子类上了，而且我们的实例也是通过父类创建出来的，也就不会出现上面的那种限制了。

我们来实现一下：

```javascript
//先来创建子类的构造函数
function MyDate() {}
MyDate.prototype.log = function () {
    return this.getDate();
}

var time = new Date(); //创建父类实例
Object.setPrototypeOf(time, MyDate.prototype); //实例的__proto__指向子类原型对象
Object.setPrototypeOf(MyDate.prototype, Date.prototype); //子类原型对象指向父类原型对象

console.log(time.log());
```

这个方法其实也有缺点，根据MDN文档，`Object.setPrototypeOf()`方法是十分浪费性能的，所以除非迫不得已，还是少用这种方法。

### 拓展

关于JavaScript中的面向对象我们已经讲完了，下面就来尝试实现几个原生的API：new，Object.create()

#### new

new是一个关键字，我们自然不能创建一个关键字，我们这里就创建一个new函数：

```javascript
function MyNew() {
    //注意，这个实现没有进行错误处理，仅考虑了参数合理的情况,为了理解new已经够了
    
    var obj = {};
    //获取构造函数和参数
    var Cons = [].shift.call(arguments); //这个操作有的人可能看的有点晕，
    								  //这么写是因为arguments其实不是一个数组
    								  //它仅有一个length属性，所以没有数组的方法。
    //将实例对象的__proto__属性指向构造函数的prototype属性
    obj.__proto__ = Cons.prototype;
    //调用构造函数
    Cons.apply(obj, arguments);
    return obj;
}
```

我们来加上一些代码，测试一下：

```javascript
function Animal(name) {
    this.name = name;
}
Animal.prototype.eat = function (food) {
    console.log(this.name + " is eating " + food);
}

//测试一下
var kitty = MyNew(Animal, "kitty");
console.log(kitty.name); //console: kitty
kitty.eat("fish"); //console: kitty is eating fish
```

#### Object.create()

```javascript
Object.prototype.myCreate = function (proto) {
    //注意，这里也省略了错误处理，并且真实的create方法有第二个参数

    function F() {}; //创建一个空的构造函数
    F.prototype = proto; //将该函数的prototype指向属性proto（传入的原型对象）
    return new F(); //返回的对象的__proto__根据F构造函数的prototype设置，
    //因此返回一个仅有__proto__属性(指向传入的原型对象)的空对象。
}

//测试
console.log(Object.myCreate(Date.prototype)); //在浏览器打印出来得到要一个对象
										  //对象的__proto__指向Date原型对象。
```

***

参考资料：

[MDN-JavaScript对象](https://developer.mozilla.org/zh-CN/docs/Learn/JavaScript/Objects)

InfoQ * Interview Map 《前端面试指南》