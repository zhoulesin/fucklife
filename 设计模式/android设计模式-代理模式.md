https://www.jianshu.com/p/9b1fd124c881

## 静态代理

### 基本概念

```txt
目标对象为其他对象提供一种代理对象,其他对象通过代理对象来控制对目标对象的访问
```

### 角色划分

```txt
- Proxy:代理对象,其他对象直接控制的对象
- Subject:目标接口,目标对象的抽象
- RealSubject:具体目标对象,目标接口的实现,即真正控制的对象
```

### 简单理解

当其他对象对某个目标对象进行操作且需要对目标对象进行更改时,若目标对象被多个对象引用,或目标对象不可以更改,此时使用代理模式,通过对代理对象的修改,而完成最终的业务需求.

### 案例

通过代购,买了一台iphone

```txt
买港版iphone,在大陆不能直接买,找到代购公司,帮我买一个
```

```txt
目标接口:买手机的动作,行为抽象
目标对象:我->买手机的人->买手机的行为
代理对象:代购的人->代理买手机
其他对象:调用购买手机的对象
```

#### 案例代码实现

1.将行为抽象,实现目标接口

```txt
目标接口:IShopPhone
```

2.实现行为,购买

```txt
目标对象:ShopPhone
特点:实现目标接口
```

3.实现代理对象,改变由代理对象实现

```txt
代理对象:PhoneProxy
	特点1:实现目标接口
	特点2:持有目标对象的引用
```

#### 源码展现

```java
public interface IShopPhone{
    void shopPhone(String str);
}

public class ShopPhone implements IShopPhone{
    public void shopPhone(String str){
        sout(str);
    }
}

public class PhoneProxy implements IShopPhone{
    IShopPhone mTarget;
    
    public ProxyShopPhone(IShopPhone target){
        this.mTarget = target;
    }
    
    public void shopPhone(String str){
        sout("start");
        mTarget.shopPhone(str);
        sout("end");
    }
}


public class Test{
    public static void main(String[] args){
        ShopPhone shopPhone = new ShopPhone();
        shopPhone.shopPhone("直接在大陆买");
        
        PhoneProxy proxy = new PhoneProxy(shopPhone);
        proxy.shopPhone("代购,买港版");
        
    }
}

```

## 动态代理

### 与静态代理的区别

```txt
动态创建代理类(虚拟机->框架.系统帮助我们来完成创建过程)
```

### 动态代理的特点

```txt
1.代理对象不需要实现接口
2.不许她哟自己实现代理对象,有虚拟机动态生成,内部通过java反射实现
3.动态代理也叫做jdk代理或接口代理
```

### 代码实现

将动态代理中,不适用静态代理中的代理对象,通过jdk中自带的代理方法实现动态代理

```java
public class Main{
    public static void main(String[] args){
        ShopPhone shopPhone = new ShopPhone();
        shopPhone.shopPhone("直接大陆买");
        
        IShopPhone proxy = (IShopPhone)Proxy.newProxyInstance(shopPhone.getClass().getClassLoader(),
                                                 shopPhone.getClass().getInterfaces(),
                  new InvocationHandler(){
                      public Object invoke(Object proxy,Method method,Object[] args){
                          sout("start");
                          Objext value = method.invoke(shopPhone,args);
                          sout("end");
                          return value;
                      }                                
                  });
        
        proxy.shopPhone("代购,买港版");
    }
}
```

- newProxyInstance详解:

  ClassLoader loader:指定当前目标对象使用的类加载器,及目标对象的类加载器

  interfaces:目标对象实现的接口的类型,使用泛型方式确认类型

  invokationHandler:事件处理,执行目标对象方法时,会触发事件处理器的方法,会把当前执行目标对象的方法作为参数传入.即动态代理接口的实现.

```java
public class ProxyFactory{
    private Object mTarget;
    
    public ProxyFactory(Object target){
        this.mTarget = target;
    }
    
    public Object getProxyInstance(){
        return Proxy.newProxyInstance(mTarget.getClass().getClassLoader(),
                                      mTarget.getClass().getInterfaces(),
                   new InvokcationHandler(){
                       public void invoke(Object proxy,Method method,Object[] args){
                           sout("start");
                           Object value = method.invoke(mTarget,args);
                           sout("end");
                           return value;
                       }                       
                   })
    }
    
    
}
```

