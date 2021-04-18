>本文是对王争大佬《设计模式之美》的学习总结

## 里式替换原则 Liskov Substitution Principle (LSP)

&emsp;&emsp;里式替换原则的英文翻译是：**Liskov Substitution Principle**，缩写是**LSP**。这个原则最早是在1986年有Barbara Liskov提出，他是这么描述这条原则的：**If S is a subtype of T,then objects of type T may be replaced with objects of type S,without breaking the program.** 在1996年，**Robert Martin**在他的**SOLID**原则中，重新描述了这个原则，英文原话是这样的：**Functions that use pointers of references to base classes must be able to use objects of derived classes without knowing it.**

&emsp;&emsp;综合两者的描述，这条原则的中文描述是这样的：子类对象（**object of subtype/derived class**）能够替换程序（**program**）中父类对象（**object of base/parrent class**）出现的任何地方，并且保证原来程序的逻辑行为（**behavior**）不变及正确性不被破坏。

&emsp;&emsp;虽然从定义描述喝代码实现上来看，多态和里式替换原则有点类似，但它们关注的角度是不一样的。多态是面向对象编程的一大特性，也是面向对象编程语言的一种语法。它是一种代码实现的思路。而里式替换原则是一种设计原则，是用来指导继承关系中子类该如何设计的，子类的设计要保证在替换父类的时候，不改变原有程序的逻辑及不破坏原有程序的正确性。

## 哪些代码明显违背了LSP？

&emsp;&emsp;实际上，里式替换原则还有另外一个更加能落地、更有指导意义的描述，那就是“**Design By Contract**”，中文翻译就是“按照协议来设计”。进一步解读下就是，子类在设计的时候，要遵守父类的行为约定（或者叫协议）。父类定义了函数的行为约定，那子类可以改变函数的内部实现逻辑，但不能改变函数原有的行为约定。这里的行为约定包括：函数声明要实现的功能；对输入、输出、异常的约定；甚至包括注释中所罗列的任何特殊说明。实际上，定义中父类和子类之间的关系，也可以替换成接口和实现类之间的关系。

举几个违反里式替换原则的例子：

### 1.子类违背父类声明要实现的功能

&emsp;&emsp;父类中提供的`sortOrdersByAmount()`订单排序函数，是按照金额从小到大来给订单排序的，而子类重写这个方法之后，是按照创建日期来给订单排序的。那子类的设计就违背了里式替换原则。

### 2.子类违背父类对输入、输出、异常的约定

&emsp;&emsp;在父类中，某个函数约定：运行出错的时候返回 **null**；获取数据为空的时候返回空集合（**empty collection**）。而子类重载函数之后，实现变了，运行出错时返回异常（**exception**），获取不到数据返回 **null**。那子类的设计就违背了里式替换原则。

### 3.子类违背父类注释中所罗列的任何特殊说明