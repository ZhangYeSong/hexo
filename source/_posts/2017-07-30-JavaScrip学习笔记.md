---
title: JavaScript学习笔记
date: 2017-07-30 16:12:45
categories:
 - JavaScript
tags:
 - JavaScript
---
> 本文为本人原创，转载请注明作者和出处。
另：本文针对的是ECMAScript 5标准。

> 学习编程有这么一种说法，最难学的是你的第二门编程语言。因为第一门编程语言的语法、思维乃至内在的思想内核深深地影响着你学习第二门语言。

正在学习JavaScript的我对这样的说法大致赞同。虽然最早学过c语言，但Java才是我的第一门掌握的比较深入的语言。Java和JavaScript看着名字挺像，但实际上是差别挺大的两门语言，并且JavaScript和其他基于类的面向对象语言也有很大不同。下面会记录一下作为一个Java程序员在学习JavaScrip的时候觉得比较关键的地方。

### 变量
Java是强类型语言，变量都是强类型的，而JavaScript是弱类型语言，变量没有类型。这是两者最大的不同点之一。都说习惯了Java这种强类型语言的人开始使用弱类型语言会有种从如释重负的感觉，但我想大多数人会觉得很不习惯而不是很爽。尽管在JavaScript中，一个变量可以保存任意类型的数据，但我们还是尽量不要随意去改变一个变量的数据类型。

### 数据类型
JavaScript包含五种基本数据类型和一种引用数据类型。基本数据类型为：number，boolean，string，null，undefined；引用数据类型为object。

很多像Java这样面向对象的语言把string当作引用数据类型，但在JavaScript中它和number一样是基本数据类型，这样似乎更符合人的思维方式？JavaScript中的string既可以用双引号括起来，可以用单引号。

由于JavaScript中声明对象的时候并不会指定对象的类型，因此需要undefined这种数据类型来代表一个变量还没有被定义类型。

通过typeof关键字可以来查看一个变量的数据类型。对于引用数据类型，和Java一样通过instanceof关键字可以查看其“类型”。

### 循环语句
JavaScript支持以下四种循环语句：

- **for** - 循环代码块一定的次数
- **for/in** - 循环遍历对象的属性
- **while** - 当指定的条件为 true 时循环指定的代码块
- **do/while** - 同样当指定的条件为 true 时循环指定的代码块

其他都没什么特别，for/in和Java中后来的foreach循环一样，同样是对数组中每一个元素进行操作。

```
for (property in window) {
  document.write(property+"<br/>")
}
```

如上的代码就是遍历window对象中的每一个属性，大家可以看看window对象里有多少属性哦。

### 方法
在JavaScript中方法实际上也是一种对象，而不像Java中方法只是对象的功能。在JavaScript中没有方法重载，但不管在定义方法的时候有几个参数，调用参数的时候可以传递任意个参数。这是因为所有参数传进来时是被当作一个数组，这个数组的名字就是arguments。虽然不支持重载，但我们可以通过如下方法来实现函数重载：

```
function add(){
  if(arguments.length == 2) {
    return arguments[0] + arguments[1];
  } else if(arguments.length == 3){
    return arguments[0] + arguments[1] + arguments[2];
  }
}
alert(add(1,2));//输出3
alert(add(1,2,3));/／输出6
```

### 块级作用域
JavaScript中是没有块级作用域这一概念的。而在有这一概念的语言中，if和for语句，代码块中的代码执行完之后，在其中定义的变量就会被回收而无法使用。在JavaScript中不会出现这样的情况。这也是需要注意的一点。

### 创建类型和对象
JavaScript没有类的概念，如何去自定义类型并创建对象呢？工厂模式是一种方法，但工厂模式得到的对象都是直接继承Object，彼此直接没有关系。因此可以使用自定义构造函数来批量创建一类对象。

```
function Person(name, age) {
    this.name = name;
    this.age = age;
}

Person.prototype = {
    constructor : Person,
    sayName : function() {
        alert(this.name);
    }
}

var person1 = new Person("Jack",18);
var person2 = new Person("Rose",16);
document.write(person1.name+":"+person1.age);
document.write(person2.name+":"+person2.age);
person1.sayName();
person2.sayName();
```

来看一段示例代码。这段代码组合使用了构造函数模式和原型模式，构造函数定义实例属性，原型模式定义方法和共同的属性。这也是现在js最常见的自定义类型的方式。

先看构造函数，几乎与Java中的构造函数一模一样，不需要return，类型名首字母大写。在构造函数中也是可以定义函数的，但是在JavaScript中函数也是对象，为了避免重复创建对象，因此把函数放到外部或者原型中是更好地选择。

再来看原型模式，其实就是定义Person的prototype属性，定义构造函数为上面的Person，并定义一个公共的方法sayName。prototype中的属性方法是所有实例化共有的，因此相当于Java中加了static的属性和方法。和static关键字一对比的话，就很容易理解了。

学了创建类型和对象之后，我依然感到很惆怅，继承和方法重写怎么办？而且JavaScript还没有方法重载。以前学Java的时候只知道封装、继承、多态是面向对象的基础，但不明白为什么它们为何如此重要。学了新的面向对象语言之后，对封装、继承、多态这些理解更深了。

### 继承

越学越觉得JavaScript比较奇怪，无怪它被称之为最让人“misunderstanding”的语言之一了。身为面向对象语言却居然连继承的关键词都没有。那么JavaScript是如何实现继承的呢————通过原型链。

```
function Person(name, age) {
    this.name = name;
    this.age = age;
}

Person.prototype = {
    constructor : Person,
    sayName : function() {
        alert(this.name);
    }
}

function Employee(name, age, salary) {
    this.name = name;
    this.age = age;
    this.salary = salary;
}

Employee.prototype = new Person();
Employee.prototype.sayName = function() {
    alert("I'm a worker name of " + this.name)
}

var employee = new Employee("zhangsan", 23, 12000);
document.write(employee.name+":"+employee.age+":"+employee.salary);
employee.sayName();
```

这里定义一个Employee类型继承于Person对象，增加了属性salary，并重写了sayName。其他的还好理解，之一Employee.prototype = new Person()这一句实现继承的关键代码有点费解。

原型链的基本思想是让一个引用类型继承另一个引用类型的属性和方法。每个构造函数都有一个原型对象，原型对象都包含一个指向构造函数的指针，这里我们让Employee的原型对象等于一个Person的对象的实例。此时Employee的原型对象就包含了指向Person原型对象的指针，Person的原型也包含了指向Employee构造函数的指针。如果Person也继承例如Animal这样的类型，这个原型的指向关系就会层层递进，形成实例与原型的链条，从而在JavaScript中实现继承。

当然JavaScript中还有借用构造函数、组合继承的方式实现继承，这里就不多写了。