---
title: JAVA代理学习
tags: [java]
index_img: https://blog-img-1304606000.cos.ap-beijing.myqcloud.com/img/20220127195018.png
---



# JAVA的三种代理学习

## 1 什么是代理

  代理(Proxy)是一种设计模式,定义：为其他对象提供一个代理以控制对某个对象的访问，即通过代理对象访问目标对象.这样做的好处是:可以在目标对象实现的基础上,增强额外的功能操作,即扩展目标对象的功能。

![proxy](https://blog-img-1304606000.file.myqcloud.com/img/proxy.png)

## 2  JAVA中的三种代理

### 2.1 静态代理

  代理对象与被代理对象需要继承共同父类或实现相同接口，特点是代理类在编译期就已经确定，即编译器就已经存在它的class文件。

#### 代码实例

1. 创建一个接口，代理类和被代理类要实现该接口。

   ```java
   public interface Hello {
   
       void hello();
   }
   ```

2. 创建被代理类

   ```java
   public class HelloService implements Hello{
       @Override
       public void hello() {
           System.out.println("Hello World!");
       }
   }
   ```

3. 创建代理类，其中保存被代理的对象。希望在执行被代理类方法前后执行一些其他逻辑。

   ```java
   public class HelloServiceProxy implements Hello{
   
       private HelloService helloService;
   
       public HelloServiceProxy(HelloService helloService) {
           this.helloService = helloService;
       }
   
       @Override
       public void hello() {
           System.out.println("你好，世界");
           helloService.hello();
           System.out.println("Saluton mondo");
       }
   }
   ```

4. 测试

   ```java
   public class StaticProxy {
   
       public static void main(String[] args) {
           Hello myHello = new HelloServiceProxy(new HelloService());
           myHello.hello();
       }
   }
   ```
   
   输出：
   
   ![image-20211210121726477](https://blog-img-1304606000.cos.ap-beijing.myqcloud.com/img/image-20211210121726477.png)
   
   可以看到通过接口调用方法，实际上执行的是代理类的方法，而代理类的方法中又执行了目标类的方法。

  静态代理做到了在不修改目标类的情况下增强了目标类的方法，但是对每一个目标类都要创建相应的代理类，代理类要继承公共父类或实现公共接口，同时还要定义相同的方法，代码量有点多，且存在不少冗余。下面的动态代理就解决了静态代理这些缺点。



### 2.2 JDK动态代理

  动态代理，不需要在编译期就确定代理类，而是在运行期生成代理类字节码文件，加载使用。JDK的动态代理实现要求代理类必须实现某个接口。

#### Java实现

* **生成代理类方法**

  调用java.lang.reflect包下`Proxy`类的`newProxyInstance()`方法

  ```java
  public static Object newProxyInstance(ClassLoader loader,
                                        Class<?>[] interfaces,
                                        InvocationHandler h)
  ```

  参数：`loader`用来加载代理类的类加载器，`interfaces`代理类实现接口集合，`InvocationHandler`为reflect包中的接口。

* **InvacationHandler接口**

  `InvocationHandler`为reflect包中的接口，需要实现其中`invoke`方法，在其中可以实现对目标类方法的增强，比如说在方法执行前后添加其他逻辑，调用代理类方法时其实就会调用该方法。

#### 代码实例

  Hello接口，HelloService中hello方法改变为带参数返回值为String的方法。

```java
public class HelloService implements Hello{

    @Override
    public String hello() {
        return "Hello,World!";
    }
}
```

以下是实现JDK动态代理步骤

1. 定义自己的InvacationHandler

   ```java
   public class CustomInvocationHandler implements InvocationHandler {
   
       private Object target;
   
       public CustomInvocationHandler(Object target) {
           this.target = target;
       }
   
       @Override
       public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
           System.out.println("DoSome Before");
           Object res = method.invoke(target,args);
           System.out.println("DoSome After");
           return res;
       }
       
       public Object getNewProxy() {
           return Proxy.newProxyInstance(CustomInvocationHandler.class.getClassLoader(),target.getClass().getInterfaces(),
                   this);
       }
   }
   ```

2. 测试

   ```java
   public class DynamicProxy {
   
       public static void main(String[] args) {
           Hello helloService = new HelloService();
           CustomInvocationHandler invocationHandler = new CustomInvocationHandler(helloService);
           Hello myHello  = (Hello) invocationHandler.getNewProxy();
           String hs = myHello.hello("xzz");
           System.out.println(hs);
       }
   }
   ```

   结果：

   ![image-20211210170545910](https://blog-img-1304606000.cos.ap-beijing.myqcloud.com/img/image-20211210170545910.png)

3. 总结

   通过上面的代码演示可以大概概述处JDK动态代理的过程，编译器并没有实现代理类，但是实现了InvovationHandler接口，调用Proxy类的静态方法`newProxyInstance()`方法时，会传入类加载器，以及目标类的接口信息，还有InvocationHandler的实现，之后运行时会生成代理类,对代理类相关方法的调用会转发到InvocationHandler中的`invoke`方法，在invoke方法中可以加入其他逻辑，其中通过反射调用目标类方法。

#### 原理探究

[参考博客](https://www.iteye.com/blog/rejoy-1627405)

  以上介绍了JDK的动态代理，其实现是基于接口的，原理探究中也发现代理类继承了`Proxy`类，而JAVA不支持多继承，所以也只能代理实现了接口的类，并且也只能代理接口中的方法，这是一个局限。而下面介绍的Cglib代理则是基于继承实现的，可以做到JDK动态代理所作不到的。

### 2.3 CGLIB代理

  cglib（Code Generation Library）是一个第三方代码生成类库，它可以在运行期扩展Java类与实现Java接口，基于它可以实现动态代理。Cglib是基于继承实现，所以不需要代理类必须实现接口，基于asm字节码修改技术，调用方法不用反射。

#### 代码实例

1. 添加Cglib依赖，使用Maven

    ```java
    <dependency>
        <groupId>cglib</groupId>
        <artifactId>cglib</artifactId>
        <version>3.3.0</version>
    </dependency>
    ```

2. 创建目标类，不继承父类，不实现接口。

    ```java
    public class OrderDao {
    
        public void insertOrder() {
            System.out.println("插入订单");
        }
    }
    ```

3. 创建类实现Cglib中的MethodInterceptor接口。

      实现其中intercept方法，通过MethodProxy类中invokeSuper方法可以调用被代理类的方法。

    ```java
    public class CustomInterceptor implements MethodInterceptor {
    
        @Override
        public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
            System.out.println("事务开始");
            Object res = methodProxy.invokeSuper(o,objects);
            System.out.println("事务结束");
            return res;
        }
    }
    ```


4. 测试

   ```java
   public class CglibTest {
       public static void main(String[] args) {
           //指定代理类字节码输出位置
           System.setProperty(DebuggingClassWriter.DEBUG_LOCATION_PROPERTY, "F:\\class");
           Enhancer enhancer = new Enhancer();
           //指定被代理的类
           enhancer.setSuperclass(OrderDao.class);
           
           //设置回调，会把对代理类方法调用转发至此，在此对目标类方法做增强。
           enhancer.setCallback(new CustomInterceptor());
           OrderDao orderDao = (OrderDao) enhancer.create();
           orderDao.insertOrder();
       }
   }
   ```

   结果：

   ![image-20211210210626946](https://blog-img-1304606000.cos.ap-beijing.myqcloud.com/img/image-20211210210626946.png)

5. 查看磁盘中生成的代理类

     IDEA 貌似自动装了反编译插件，java bytecode complier,看了之后看不懂。



## 3 总结

  以上就是 Java 中的三种代理，通过代理可以实现很多功能，如Spring中AOP,各种简化开发的注解，具体有待进一步学习。

