---
layout: post
title: "简洁的Scala"
date: 2013-04-05 13:47
comments: true
categories: language
published: false
---
Scala在设计时是很注重一致性(consistency)的，Scala的简单性(concision)都是由其一致性带来的。

Scala的看上去很复杂，但是它在概念上是非常一致的。弄清了几个概念后，也就不觉得复杂了，反倒是比Java的简单。

##一切都是对象
更精确地说，应该是“一切值都是对象”。

* 整数, 浮点数等基本类型也是对象  

``` scala
123.toByte
3.14.toInt
```
在Java中，primitive type不是对象，打破了一致性。

* 函数也是对象

``` scala
val compare = (x: Int, y: Int) => x > y
compare(1, 20
```

* Java static 方法和属性，有点打破了面向对象，因为它们不属于一个实例，而是属于类。在Scala中，static 方法和属性也属于对象，具体来说，属于单例对象。

``` scala
object Dog {
  val sound = "wang wang" //static field
}
```

##函数是值
函数是一等公民，跟普通的值没区别

* 可以当作参数传递

``` scala
val  compare = (x: Int , y: Int ) => x >  y
list sortWith compare
```

* 不管它是实例的方法

``` scala
class AComparator  {
  def  compare(x: Int , y: Int ) = x >  y
}
list sortWith ( new  AComparator ).compare
```

* 还是匿名子句

``` scala
object  annonymous extends scala.Function2[Int , Int , Boolean] {
  override  def  apply(x: Int , y: Int ) = x >  y
}
list sortWith annonymous 
```

##一切操作都是函数调用
* 运算符是函数调用

``` scala
1 +  1
1.+(1)
1.>(0)
1 >  0
(1 >  0).&(2 >  1)
(1 >  0) & 2 >  1
stack.push( 10)
stack push  10
stack.pop
stack pop
```
注意，上述代码中，只有一个参数或零个参数的方法在调用时可以省略”.” 和”()”。

* 更多的符号需要用作方法名

``` scala
def  !@#%^&*\-<=>?|~:/ = println("noop" )
def  √(x: Double ) = Math.sqrt( x)
val  Π =  Math.Pi
val  r =  √( 9*Π)
```

* ‘<’, ‘>’ 更适合作方法名，所以用’[’ 和‘]’ 来表示类型参数

* for语句是函数调用

``` scala
for  (i <- List(1, 2)) {
  println(i)
}
List(1, 2) foreach { i => println(i)}
for  (i <- List(1, 2))  yield {
  i +  10
}
List(1, 2) map {i => i +  10}
```
* 更多的例子

``` scala
// synchronized is function call instead of keyword
def  check = synchronized {
  // isInstanceOf is function call instead of keyword
  100.isInstanceOf[ String ] 
}
```
* 额外的好处：自左向右顺序书写语句

``` scala
stack.pop.asInstanceOf[ Int ] // (Integer) stack.pop() in Java
```

##一切操作都返回值
* 默认返回最后一条语句的值，也可以用return 显式返回

``` scala
  val  r1 = { // return 3
    val  a =  1
    val  b =  2
    a +  b
  }
  val  r2 =  if (true) 1 else 2
  val  r3 =  // return (): Unit
    for  (i <- List(1, 2)) {
      println(i)
    }
  val  r4 =  // return List(11, 12)
    for  (i <- List(1, 2))  yield {
       i +  10
    }
  val  r5 =  // return java.io.File
    try  {
       val  f =  new  File("afile")
       f
    }  catch {
       case ex: IOException  => null
    }
```
Scala里不再像C/C++, Java，区分语句(statement)和表达式(expression)。Scala里没有statement，只有expression，因此一切操作都是表达式，都有返回值。Scala可以说是expression-oriented。

关于语句和表达式的区别，可以看"Scala in depth"一书中的比较:

> **Statement Versus Expression** A statement is something that execute; an expression is something that evaluates to a value.

##一些讨论
[2 楼 dcaoyuan 2009-10-25](http://dcaoyuan.iteye.com/blog/502730)  

> 这次会上讲PPT时间有点不够，但这个PPT中提到的每句话都是仔细想过的，强调的是Scala中一些概念的一致性，比如，有关Scala的文章中常提到“值都是对象”，准确地说，应该是“值都是对象的实例”，还有，“操作、函数、参数可以互相转化，都是值，都是实例的对象”，这其实是Scala可以扩展的关键，也是OO+FP能够比较好地在Scala中结合的关键。 
> 
> 还有就是Scala的类型体系，看上去复杂，其实是为了修正Java中类型体系的一些问题，并且带了了一个JVM上的完整一致的类型体系。弄清了几个概念后，也就不觉得复杂了，反倒是比Java的简单。 
> 
> 总之，我看到的Scala的简单性都是由其一致性带来的，虽然设计者为了开发人员的习惯作了一点点妥协，但还是非常坚持他的设计理念。 
> 
> Martin不是一个简单的人，为了在JVM上实现他的设计理念，其后台的具体实现其实是相当复杂和困难的，但他坚持并不断实践，有时候甚至不惜把原来的实现推倒重来，终于在2.6以后有了我们现在看到的相当理想的结果。

##参考资料
1. [2009. 邓草原. 对Java的修正和超越](http://www.slideshare.net/dcaoyuan/scalajava)

