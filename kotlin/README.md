#kotlin教程

http://www.runoob.com/kotlin/kotlin-tutorial.html

## 第一个kotlin程序

```kotlin
package hello		//可选的包头

fun main(args:Array<String>){	//包级可见的函数,接收一个字符串数组作为参数
    println("Hello World")		//分号可以省略
}
```

- 面向对象

```kotlin
class Greeter(val name:String){
    fun greet(){
        println("Hello,$name")
    }
}

fun main(args:Array<String>){
    Greeter("World!").greet()	//创建一个对象不用new关键字
}
```

- 为什么选择kotlin
  - 简洁:大大减少样板代码的数量
  - 安全:避免空指针异常等整个类的错误
  - 互操作性:充分利用JVM,Android和浏览器的现有库
  - 工具友好:可用任何java IDE或者使用命令行构建

## 基础语法

- 包声明

  ```kotlin
  package com.zzz.main
  
  import java.util.*
  
  fun test(){}
  class Runoob{}
  ```

  kotlin源文件不需要相匹配的目录和包,源文件可以放在任何文件目录.

  以上例子中test()的全名是com.zzz.main.test,Runoob的全名是com.zzz.main.Runoob

  如果没有指定包,默认为default

- 默认声明

  有多个包会默认导入到每个kotlin文件中

  - kotlin.*
  - kotlin.annotations.*
  - kotlin.collections.*
  - kotlin.comparisons.*
  - kotlin.io.*
  - kotlin.ranges.*
  - kotlin.sequences.*
  - kotlin.text.*

- 函数定义

  函数定义使用关键字fun,参数格式为 	参数:类型

  ```kotlin
  fun sum(a:Int,b:Int):Int{	//Int参数,返回值Int
      return a+b
  }
  ```

  表达式作为函数体,返回类型自动推断:

  ```kotlin
  fun sum(a:Int,b:Int) = a+b
  
  public fun sum(a:Int,b:Int):Int = a+b	//public 方法必须明确写出返回类型
  ```

  无返回值的函数(类似Java中的void):

  ```kotlin
  fun printSum(a:Int,b:Int):Unit{
      print(a+b)
  }
  
  //如果是返回Unit类型,则可以省略(对于public方法也是这样:)
  public fun printSum(a:Int,b:Int){
      print(a+b)
  }
  ```

- 可变长参数函数

  函数的可变长参数可以用`vararg`关键字进行标识:

  ```kotlin
  fun vars(vararg v:Int){
      for(vt in v){
          print(vt)
      }
  }
  
  //测试
  fun main(args:Array<String>){
      vars(1,2,3,4,5)	//输出12345
  }
  ```

- lambda(匿名函数)

  lambda表达式使用实例

  ```kotlin
  //测试
  fun main(args:Array<String>){
      val sumLambda:(Int,Int) -> Int = {x,y -> x+y}
      println(sumLambda(1,2))	//输出3
  }
  ```

- 定义常量与变量

  可变变量定义:var关键字

  ```kotlin
  var <标识符> : <类型> = <初始化值>
  ```

  不可变量变量定义:val关键字,只能赋值一次的变量,(类似java中的final修饰的变量)

  ```kotlin
   val <标识符> :<类型> = <初始化值>
  ```

  常量与变量都可以没有初始化值,但在引用之前必须初始化

  编译器支持自动类型判断,即声明时可以不指定类型,由编译器判断

  ```kotlin
  val a: Int = 1
  val b = 1		//系统自动推断变量类型为Int
  val c : Int		//如果不在声明时初始化则必须提供变量类型
  c = 1			//明确赋值
  
  var x = 5		//系统自动推断变量类型为Int
  x += 1			//变量可修改
  ```

- 注释

  Kotlin支持单行和多行注释

  ```kotlin
  // 这是单行注释
  
  /*	这是多行的
  	块注释 */
  ```

- 字符串模板

  $表示一个变量名或者变量值

  $varName表示变量值

  ${varName.fun()}表示变量的方法返回值

  ```kotlin
  var a = 1
  //模板中的简单名称:
  val s1 = "a is $a"
  
  a = 2
  //模板中的任意表达式
  val s2 = "${s1.replace("is","was")},but now is $a"
  ```

- NULL检查机制

  Kotlin的空安全设计对于晟敏可为空的参数,在使用时要进行空判断处理,有两种处理方式,字段后加!!像java一样抛出空异常,另一种字段后加?可不做处理返回值为null或配合?:做空判断处理

  ```kotlin
  //类型后面加?表示可为空
  var age:String? = "23"
  //抛出空指针异常
  val ages = age!!.toInt()
  //不做处理返回null
  val ages1 = age?.toInt()
  //age为空返回-1
  val ages2 = age?.toInt() ?: -1
  ```

  当一个引用可能为null值时,对应的类型声明必须明确的标记为可为null

  当str中的字符串内容不是一个整数时,返回null

  ```kotlin
  fun parseInt(str:String):Int?{
      //...
  }
  ```

  以下实例演示如何使用一个返回值可为null的函数:

  ```kotlin
  fun main(args:Array<String>){
      if(args.size < 2){
          print("Two integers exptected")
          return
      }
      val x = parseInt(args[0])
      val y = parseInt(args[1])
      //直接使用'x*y'会导致错误,因为他们可能为null
      if(x != null && y!= null){
          print(x*y)
      }
  }
  ```

- 类型检测及自动类型转换

  我们可以使用is运算符检测一个表达式是否某类型的一个实例(类似java中isntanceOf关键字)

  ```kotlin
  fun getStringLength(obj:Any):Int?{
      if(obj is String){
          //做过类型判断之后,obj会被系统自动转换为string类型
          return obj.length
      }
      //在这里有一种写法,与java的instanceof不同,使用is
      //if(obj !is String){
      // //xxx
      //}
      
      // 这里的obj仍然是Any类型的引用
      return null
  }
  ```

  或者

  ```kotlin
  fun getStringLength(obj:Any):Int?{
      if(obj !is String)
          return null
      //在这个分支中,'obj'的类型会被自动转换为'string'
      return obj.length
  }
  ```

  甚至还可以

  ```kotlin
  fun getStringLength(obj:Any):Int?{
      //在'&&'运算符的右侧,'obj'的类型会被自动转换为'string'
      if(obj is String && obj.length >0)
      	return obj.length
      return null
  }
  ```

- 区间

  区间表达式由具有操作符形式..的rangeTo函数辅以in和!in形成

  区间是为任何可比较类型定义的,但对于整型原生类型,它有一个优化得实现.以下是使用区间的一些示例:

  ```kotlin
  for(i in 1..4) print(i) //输出1234
  
  for(i in 4..1) print(i)	//不输出
  
  if(i in 1..10){	//等同于 1<=i&&i<=10
      println(i)
  }
  
  //使用step指定步长
  for(i in 1..4 step 2) print(i) //输出13
  
  for(i in 4 downTo 1 step 2) print(i) //输出42
  
  //使用until函数排除结束元素
  for(i in 1 until 10){	//i in [1,10) 排除了10
      println(i)
  }
  ```

  实例测试

  ```kotlin
  fun main(args:Array<String>){
      print("循环输出")
      for(i in 1..4) print(i)
      println("\n--------")
      print("设置步长")
      for(i in 1..4 step 2) print(i)
      println("\n---------")
      print("使用downTo")
      for(i in 4 downTo 1 step 2) print(i)
      println("\n---------")
      print("使用until")
      for(i in 1 until 5) print(i)
      
  }
  ```

## 基本数据类型

​	Kotlin的基本数值类型,包括Byte,Short,Int,Long,Float,Double等,不同于java的是,字符不属于数值类型,是一个独立的数据类型

| 类型   | 位宽度 |
| ------ | ------ |
| Double | 64     |
| Float  | 32     |
| Long   | 64     |
| Int    | 32     |
| Short  | 16     |
| Byte   | 8      |

- 字面常量

  下面是所有类型的字面常量

  - 十进制:123
  - 长整型以大写L结尾:123L
  - 16进制以0x开头:0x0F
  - 2进制以0b开头:0b00001011
  - 注意:8进制不支持

  Kotlin同时也支持传统符号表示的浮点数值:

  - Doubles默认写法:12.5   123.5e10
  - Floats使用f或F后缀,12.5f

  你可以使用下划线使数字常量更易读:

  ```kotlin
  val oneMillion = 1_000_000
  val creditCardNumber = 1234_456_789L
  val socialSecurityNumber = 999_999_999L
  val hexBytes = 0xFF_EC_DE_5E
  val bytes = 0b11010101_10101011000_111
  ```

- 比较两个数字

  Kotlin中没有基础数据类型,只有封装的数字类型,你每定义的一个变量,其实Kotlin帮你封装了一个对象,这样可以保证不会出现空指针.数字类型也一样,所有在比较两个数字的时候,就有比较数据大小和比较两个对象是否相同的区别了.

  在kotlin中,三个等号===表示比较对象地址,两个==表示比较两个值大小

  ```kotlin
  fun main(args:Array<String>){
      val a:Int = 10000
      println(a === a)	//true值相等,对象地址也相等
      
      //经过了装箱,创建了两个不同的对象
      val boxedA:Int? = a
      val anotherBoxedA:Int? = a
      
      //虽然经过了装箱,但是值时相等的,都是10000
      println(boxedA == anotherBoxedA) true 值相等
      println(boxedA === anotherBoxedA) false 对象地址不一样
  }
  ```

- 类型转换

  由于不同的表示方式,较小类型并不是较大类型的子类型,较小的类型不能隐式转换为较大的类型.这意味着在不进行显示转换的情况下我们不能把Byte型值赋给一个Int变量

  ```kotlin
  val b:Byte =1
  val i:Int = b	//错误
  ```

  我们可以用其toInt()方法

  ```kotlin
  val b: Byte = 1
  val i : Int = b.toInt()	//OK
  ```

  每种数据类型都有下面这些方法,可以转换为其他的类型

  ```kotlin
  toByte()
  toShort()
  toInt()
  toLong()
  toFloat()
  toDouble()
  toChar()
  ```

  有些情况下也是可以使用自动类型转换的,前提是可以根据上下文环境推断出正确的数据类型而且操作符会做响应的重载

  ```kotlin
  val l = 1L + 3 //Long+Int => Long
  ```

- 位操作符

  对于Int和Long,还有一系列的位操作符可以使用

  ```kotlin
  shl(bits)	左移 <<
  shr()		右移>>
  ushr()		无符号右移 >>>
  and()		与 &
  or()		或 |
  xor()		异或 ^|
  inv()		反向^
  ```

- 字符

  和java不一样,Kotlin中的Char不能直接和数字操作,Char必须是`'`包含起来的.

  ```kotlin
  
  ```

  字符字面值用单引号括起来'1'.特殊字符可以用反斜杠转义.支持这几个转义序列:\t,\b,\n,\r,\',\",\\\,和\$.编码其他字符要用Unicode转义序列语法:'\uFF00'

  我们可以显示把字符转换为Int数字

  ```kotlin
  fun decimalDigitValue(c:Char):Int{
      if(c !in '0'..'9')
      	throw IllegalArgumentException("out of range")
      return c.toInt() - '0'.toInt() //显示转换为数字
  }
  ```

  但需要可空引用时,像数字,字符会被装箱.装箱操作不会保留同一性

- 布尔

  布尔用Boolean类型表示,他有两个值:true和false

  若需要可空引用布尔会被装箱

  内置的布尔运算有

  ```kotlin
  || 短路逻辑或
  && 短路逻辑与
  !  逻辑非
  ```

- 数组

  数组用Array实现,并且还有一个size属性及get和set方法,由于使用[]重载了get和set方法,所以我们可以通过下标很方便的获取或者设置数组对应位置的值.

  数组的创建两种方式:一种是使用函数arrayOf().另外一种是使用工厂函数.

  ```kotlin
  fun main(args:Array<String>){
      //[1,2,3]
      val a = arrayOf(1,2,3)
      //[0,2,4]
      val b = Array(3,{i -> (i*2)})
      
      //读取数组内容
      println(a[0]) 
      println(a[1])
  }
  ```

  如上所述,[]运算符代表调用成员函数get()和set()

  注意:与java不同的是,kotlin中数组是不型变的(invariant)

  除了类Array,还有ByteArray,ShortArray,IntArray,用来表示各个类型的数组,省去了装箱操作,因此效率更高,其用法同Array一样

  ```kotlin
  val x:IntArray = intArrayOf(1,2,3)
  x[0] = x[1] + x[2]
  ```

- 字符串

  和java一样,String是不可变的.方括号[]语法可以很方便的获取字符串中的某个字符,也可以通过for循环来遍历

  ```kotlin
  for(c in str){
      print(c)
  }
  ```

  Kotlin支持三个引号"""括起来的字符串,支持多行字符串

  ```kotlin
  fun main(args:Array<String>){
      val text = """
      	asds
      	222
      	dsfgg
      """
      print(text)
  }
  ```

  string可以通过trimMargin()方法来删除多余的空白

  ```kotlin
  fun main(args:Array<String>){
      val text = """
      |fdsfd
      |fsfdsf
      |dfsfdf
      """.trimMargin()
      print(text)
  }
  ```

- 字符串模板

  字符串可以包含模板表达式,即一些小段代码,会求值并把结果合并到字符传中.模板表达式以$开头,由一个简单的名字构成

  ```kotlin
  fun main(args:Array<String>){
      val i = 10
      val s = "i = $i"
      println(s)
  }
  ```

  或者用花括号括起来的任意表达式

  ```kotlin
  fun main(args:Array<String>){
      val s = "asdff"
      var str = "$s.length is ${s.length}" // "asdff.length is 6"
      println(str)
  }
  ```

  原生字符串和转义字符串内部都支持模板.如果你需要在原生字符串中表示字面值$字符,(不支持反斜杠转义),可以用如下语法

  ```kotlin
  fun main(args:Array<String>){
      val price = """
      ${'$'}9.99
      """
      print(price)
  }
  ```

  

