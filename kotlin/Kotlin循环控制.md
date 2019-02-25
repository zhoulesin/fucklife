# Kotlin循环控制

## For循环

for循环可以对任何提供迭代器iterator的对象进行遍历

```kotlin
for(item in collection) print(item)
```

循环体可以是代码块 

```kotlin
for(item in collection){
    //...
}
```

如上所述,for可以循环遍历任何提供了迭代器的对象,

如果你想要通过索引遍历一个数组或者一个list,你可以这么做

```kotlin
for(i in array.indices){
    print(array[i])
}
```

注意这种在区间上遍历会编译成优化得实现而不会创建额外对象

或者你可以用库函数withIndex

```kotlin
for((index,value) in array.withIndex){
    print("the element at $index is $value")
}
```

## 实例

对集合进行迭代

```kotlin
fun main(args:Array<String>){
    val items = listOf("apple","banana","kiwi")
    for(item in items){
        println(item)
    }
    
    for(index in items.indices){
        println("item at $index is ${items[index]}")
    }
}
```

## while与do ... while循环

while是最基本的循环

```kotlin
while(布尔表达式){
    //afdss
}
```

对于while语句而言,如果不满足条件,则不能进入循环

do..while和while相似,但do..while至少执行一次

```kotlin
do{
    
}while(布尔表达式);
```

## 实例

```kotlin
fun main(args:Array<String>){
    println("----while使用----")
    var x = 5
    while(x>0){
        print(x--)
    }
    
    println("----do..while使用----")
    var y = 5
    do{
        println(y--)
    }while(y>0)
}
```

## 返回和跳转

Kotlin有三种结构化跳转表达式

- return.默认从最直接包围它的函数或者匿名函数返回
- break.最终最直接包围它的循环
- continue.继续下一次最直接包围它的循环

在循环中kotlin支持传统的break和continue操作符

```kotlin
fun main(args:Array<String>){
    for(i in 1..10){
        if(i==3)continue
        println(i)
        if(i>5)break
    }
}
```

## break和continue标签

在Kotlin中任何表达式都可以用标签label来标记。标签的格式为标识符后跟@符号,例如:abc@,fooBar@都是有效的标签.要为一个表达式加标签,我们只要在其前加标签即可.

```kotlin
loop@ for(i in 1..100){
    //...
}
```

现在,我们可以用标签限制break或者continue:

```kotlin
loop@ for(i in 1..100){
    for(j in 1..100){
        if(...) break@loop
    }
}
```

标签限制的break跳转到刚好位于该标签指定的循环后面的执行点.continue继续标签指定的循环的下一次迭代.

## 标签处返回

kotlin有函数字面量,局部函数和对象表达式.因此kotlin的函数可以被嵌套.标签限制的return允许我们从外层函数返回.最重要的一个用途就是从lambda表达式中返回.

```kotlin
fun foo(){
    ints.forEach{
        if(i == 0) return
        print(it)
    }
}
```

这个return表达式从最直接包围它的函数即foo中返回.(注意,这种非局部的返回只支持传给内联函数的lambda表达式.)如果我们需要从lambda表达式中返回,我们必须给它加标签并用以限制return

```kotlin
fun foo(){
    ints.forEach lit@{
        if(it==0) return@lit
        print(it)
    }
}
```

现在,它只会从lambda表达式中返回.通常情况下使用隐式标签更方便.该标签与接受该lambda的函数同名.

```kotlin
fun foo(){
    ints.forEach{
        if(it == 0) return@forEach
        print(it)
    }
}
```

或者我们用一个匿名函数替代lambda表达式.匿名函数内部的return语句将从该匿名函数自身返回

```kotlin
fun foo(){
    ints.forEach(fun(value:Int){
        if(value == 0) return
        print(value)
    })
}
```

当要返回一个值得时候,解析器优先选用标签限制的return

```kotlin
return@a 1
```

