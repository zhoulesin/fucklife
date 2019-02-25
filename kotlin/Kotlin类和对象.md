# Kotlin类和对象

## 类定义

Kotlin类可以包含:构造函数和初始化代码块,函数,属性,内部类,对象声明

Kotlin中使用关键字class声明类,后面紧跟类名:

```kotlin
class Runoob{
    
}
```

我们可以定义一个空类

```kotlin
class Empty
```

可以在类中定义成员函数

```kotlin
class Runoob{
    fun foo(){print("Foo")}	//成员函数
}
```

## 类的属性

### 属性定义

类的属性可以用关键字var声明为可变的,否则使用只读关键字val声明为不可变

```kotlin
class Runoob{
    var name:String = ...
    var url:String = ...
    var city:String = ...
}
```

我们可以像使用普通函数那样使用构造函数创建类实例:

```kotlin
val site = Runoob()	//Kotlin中没有new关键字
```

要使用一个属性,只要用名称引用它即可

```kotlin
site.name		//使用.号来引用
site.url
```

Kotlin中的类可以有一个主构造器,以及一个或多个次构造器,主构造器是类头部的一部分,位于类名称之后,

```kotlin
class Person constructor(firstName:String){}
```

如果主构造器没有任何注解,也没有任何可见度修饰符,那么constructor关键字可以省略

```kotlin
class Person(firstName:String){}
```

### getter和setter

属性声明的完整语法:

```kotlin
var <propertyName>[:<PropertyType>][=<property_initializer>]
	[<getter>]
	[<setter>]
```

getter和setter都是可选

如果属性类型可以从初始化语句或者类的成员函数中推断出来,那就可以省去类型,val不允许设置setter函数,因为它是只读的,

```kotlin
var allByDefault:Int?	//错误:需要一个初始化语句,默认实现了getter和setter方法
var initialized = 1		//类型为Int,默认实现了getter和setter
val simple:Int?			//类型为Int,默认实现了getter,但必须在构造函数中初始化
val inferredType = 1	//类型为Int,默认实现getter
```

### 实例

以下实例定义了一个Person类,包含两个可变变量lastName,和no,,,,,lastName修改了getter方法,no修改了setter方法

```kotlin
class Person{
    var lastName:String = "zhang"
    	get() = field.toUpperCase()	//将变量赋值后转换为答谢
    	set
    
    var no:Int = 100
    	get() = field		//后端变量 
    set(value){
        if(value < 10){
            field = value	//如果传入的值小于10 返回该值
        }else{
            field = -1		//如果传入的值大于等于10,返回-1
        }
    }
    
    var height:Float = 145.4f
    	private set
}

//测试
fun main(args:Array<String>){
    var person:Person = Person()
    
    person.lastName = "wang"
    
    println("lastName:${person.lastName}")
    
    person.no = 9
    println("no:${person.no}")
    
    person.no = 20
    println("no:${person.no}")
}
```

Kotlin中类不能有字段.提供了Backing FIelds后端变量机制,备用字段使用field关键字声明,field关键词只能用于属性的访问器,如以上实例:

非空属性必须在定义的时候初始化,kotlin提供了一种可以延迟初始化的方案,使用lateinit关键字描述属性:

```kotlin
public class MyTest{
    lateinit var subject:TestSubject
    
    @SetUp fun setup(){
        subject = TestSubject()
    }
    
    @Test fun test(){
        subject.method()
    }
}
```

## 主构造器

主构造器中不能包含任何代码,初始化代码可以放在初始化代码段中,初始化代码段使用init关键字作为前缀

```kotlin
class Person constructor(firstName:String){
    init{
        println("FirstName is firstName")
    }
}
```

注意:主构造器的参数可以在初始化代码段中使用,也可以在类主题n定义的属性初始化代码中使用.一种简介语法,可以通过主构造器来定义属性或初始化属性值

```kotlin
class People(val firstName:String,val lastName:String){
    //...
}
```

如果构造器有注解,或者有可见度修饰符,这时constructor关键字是必须的,注解和修饰符要放在它之前

### 实例

```kotlin
class AAA constructor(name:String){
    var url:String = "http://www.baidu.com"
    var country:String = "CN"
    var siteName = name
    
    init{
        println("初始化网站名:${name}")
    }
    
    fun printTest(){
        println("我是类的函数")
    }
}

fun main(args:Array<String>){
    val run = AAA("asds")
    println(run.siteName)
    println(run.url)
    println(run.country)
    run.printTest()
}
```

## 次构造函数

类也可以由二级构造函数,需要加前缀 constructor

```kotlin
class Person{
    constructor(parent:Person){
        parent.children.add(this)
    }
}
```

如果类有主构造函数,每个次构造函数都要或直接或间接通过另一个次构造函数代理主构造函数..在同一个类中代理另一个构造函数使用this关键字:

```kotlin
class Person(val name:String){
    constrcutor(name:String,age:Int) :this(name){
        //..初始化
    }
}
```

如果一个非抽象类没有声明构造函数(主构造函数或次构造函数),他会产生一个没有参数的构造函数.构造函数是public,如果你不想你的类有公共的构造函数,你就得声明一个空的主构造函数:

```kotlin
class DontCreateMe private constructor(){}
```

`在jvm虚拟机中,如果主构造函数的所有参数都有默认值,编译器会生成一个附加的无参的构造函数,这个构造函数会直接使用默认值..这使得kotlin可以更简单的使用像jackson或者JPA这样使用无参构造函数来创建类实例的库`

```kotlin
class Customer(val customerName:String = "")
```



