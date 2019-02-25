# Kotlin条件控制

## IF表达式

一个if语句包含一个布尔表达式和一条或多条语句

```kotlin
//传统用法
var max = a
if(a < b) max = b

//使用else
var max:Int
if(a>b){
    max = a
}else{
    max = b
}

//作为表达式
val max = if(a<b) a else b
```

我们也可以把IF表达式的结果赋值给一个变量

```kotlin
val max = if(a>b){
    a
}else{
    b
}
```

这也说明我也不需要像java那种有一个三元操作符,因为我们可以使用它来简单实现:

```kotlin
val c = if(condition) a else b
```

## 实例

```kotlin
fun main(args:Array<String>){
    var x = 0
    if(x>0){
        //aa
    }else if(x==0){
        //bb
    }else{
        //ccc
    }
    
    
    var a = 1
    var b = 2
    var c = if(a>=b) a else b
    
}
```

## 使用区间

使用in运算符来检测某个数字是否在指定区间内,区间格式为x..y

```kotlin
fun main(args:Array<String>){
    val x = 5
    val y = 9
    if(x in 1..8){
        //asdffff
    }
}
```

## When表达式

when将它的参数和所有的分支条件顺序比较,知道某个分支满足条件

when既可以被当做表达式使用也可以被当做语句使用.如果它被当做表达式,符合条件的分支的值就是整个表达式的值,如果当做语句使用,则忽略个别分支的值

when类似其他语言的switch操作符

```kotlin
when(x){
    1 -> print("1")
    2 -> print("2")
    else -> {	//注意这个块
        print("x 不是1,也不是2")
    }
}
```

在when中,else同switch的default..如果其他分支都不满足条件将会求值else分支.

如果很多分支需要用到相同的方式处理,则可以把多个分支条件放在一起,用逗号分隔

```kotlin
when(x){
    0,1 -> print("x ==0 or x == 1")
    else -> {
        print("otherwise")
    }
}
```

我们也可以检测一个值在in或者不在!in一个区间或者集合内:

```kotlin
when(x){
    in 1..10 -> print("x is in the range")
    in validNumbers -> print("x is valid")
    !in 10..20 -> print("x is outside the range")
    else -> print("none of the above")
}
```

另一种可能性是检测一个值是is或者不是!is一个特定类型的值.

ps:由于只能转换,你可以访问该类型的方法和属性,而无需任何额外的检测

```kotlin
fun hasPrefix(x:Any) = when(x){
    is String -> x.startsWith("prefix")
    else -> false
}
```

when也可以用来取代if-else链.如果不提供参数,所有的分支条件都是最简单的布尔表达式,而当一个分支的条件为真时则执行该分支

```kotlin
when{
    x.isOdd() -> print("x is odd")
    x.isEven() -> print("x is even")
    else -> print("x is funny")
}
```

## 实例

```kotlin
fun main(args:Array<String>){
    var x = 0
    when(x){
        0,1 -> println("x == 0 or x == 1")
        else -> println("otherwise")
    }
    
    when(x){
        1 -> println("x == 1")
        2 -> println("x == 2")
        else -> {
            println("x 不是1 ,也不是2")
        }
    }
    
    when(x){
        in 0..10 -> println("x 在该区间范围内")
        else -> println("x 不在该区间范围内")
    }
}
```

when中使用in运算符来判断集合内是否包含某实例

```kotlin
fun main(args:Array<String>){
    val items = setOf("apple","banana","kiwi")
    when{
        "orange" in items -> println("juicy")
        "apple" in items -> println("apple is fine too")
    }
}
```

