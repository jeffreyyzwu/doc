# Aspectj使用小结

## Aspect Oriented Programming（AOP）

> 　　面向切面编程（AOP是Aspect Oriented Program的首字母缩写），我们知道，面向对象的特点是继承、多态和封装。而封装就要求将功能分散到不同的对象中去，这在软件设计中往往称为职责分配。实际上也就是说，让不同的类设计不同的方法。这样代码就分散到一个个的类中去了。这样做的好处是降低了代码的复杂程度，使类可重用。
> 　　但是人们也发现，在分散代码的同时，也增加了代码的重复性。什么意思呢？比如说，我们在两个类中，可能都需要在每个方法中做日志。按面向对象的设计方法，我们就必须在两个类的方法中都加入日志的内容。也许他们是完全相同的，但就是因为面向对象的设计让类与类之间无法联系，而不能将这些重复的代码统一起来。
> 　　也许有人会说，那好办啊，我们可以将这段代码写在一个独立的类独立的方法里，然后再在这两个类中调用。但是，这样一来，这两个类跟我们上面提到的独立的类就有耦合了，它的改变会影响这两个类。那么，有没有什么办法，能让我们在需要的时候，随意地加入代码呢？**这种在运行时，动态地将代码切入到类的指定方法、指定位置上的编程思想就是面向切面的编程。**
> 　　一般而言，我们管切入到指定类指定方法的代码片段称为切面，而切入到哪些类、哪些方法则叫切入点。有了AOP，我们就可以把几个类共有的代码，抽取到一个切片中，等到需要时再切入对象中去，从而改变其原有的行为。这样看来，AOP其实只是OOP的补充而已。OOP从横向上区分出一个个的类来，而AOP则从纵向上向对象中加入特定的代码。有了AOP，OOP变得立体了。如果加上时间维度，AOP使OOP由原来的二维变为三维了，由平面变成立体了。从技术上来说，AOP基本上是通过代理机制实现的。
> 　　AOP在编程历史上可以说是里程碑式的，对OOP编程是一种十分有益的补充。

其他相关知识请自行google

## Aspectj介绍

[Aspectj官网](http://www.eclipse.org/aspectj/)

> + a seamless aspect-oriented extension to the Javatm programming language（一种基于Java平台的面向切面编程的语言）
> + Java platform compatible（兼容Java平台，可以无缝扩展）
> + easy to learn and use（易学易用）

## Aspectj作用

> + clean modularization of crosscutting concerns, such as error checking and handling, synchronization, context-sensitive behavior, performance optimizations, monitoring and logging, debugging support, and multi-object protocols。

　　大意是说：干净的模块化横切关注点（也就是说单纯，基本上无侵入），如错误检查和处理，同步，上下文敏感的行为，性能优化，监控和记录，调试支持，多目标的协议。

## 名词解释

| 概念        | 解释                                       |
| :-------- | :--------------------------------------- |
| pointcut  | 是一个（组）基于正则表达式的表达式，通常会选取程序中的某些我们感兴趣的执行点，或者说是程序执行点的集合。 |
| joinPoint | 通过pointcut选取出来的集合中的具体的一个执行点。             |
| Advice    | 在选取出来的JoinPoint上要执行的操作、逻辑。包含After、AfterReturning、AfterThrowing、Before、Around。 |
| aspect    | 是我们关注点的模块化，可能会横切多个对象和模块。它是一个抽象的概念，指在应用程序不同模块中的某一个领域或方面。由pointcut和advice组成。 |
| Target    | 被aspectj横切的对象。我们所说的joinPoint就是Target的某一行，如方法开始执行的地方、方法类调用某个其他方法的代码。 |

## 项目配置

* 在pom.xml中增加以下配置

    ```xml
      <dependency>
        <groupId>org.aspectj</groupId>
        <artifactId>aspectjrt</artifactId>
        <version>1.8.9</version>
      </dependency>
      <dependency>
        <groupId>org.aspectj</groupId>
        <artifactId>aspectjweaver</artifactId>
        <version>1.8.9</version>
      </dependency>
    ```

    ```xml
      <plugin>
        <groupId>org.codehaus.mojo</groupId>
        <artifactId>aspectj-maven-plugin</artifactId>
        <version>1.10</version>
        <configuration>
          <complianceLevel>1.8</complianceLevel>
          <source>1.8</source>
          <target>1.8</target>
        </configuration>
        <executions>
          <execution>
            <goals>
              <goal>compile</goal>
              <goal>test-compile</goal>
            </goals>
          </execution>
        </executions>
      </plugin>
    ```

## Aspectj语法
  这个内容比较多，可重点阅读下pointcut syntax。相关链接如下
  > + [Aspectj语法](http://www.jianshu.com/p/691acc98c0b8)

## 使用讲解

* 先上原始代码

  ```java
    //Car.java
    public void run(int speed){
        System.out.println(String.format("car is running at %dkm/h", speed));
    }
  ```

  ```java
    //AspectLog.java
    import org.aspectj.lang.annotation.*;

    @Aspect
    public class AspectLog {
        //定义切入点方法
        @Pointcut("execution(* Car.run(..))")
        private void logPointCut(){
        }
        @Before("logPointCut()")
        public void beforeLog()
        {
            System.out.println("before log ....");
        }
        @After("logPointCut()")
        public void afterLog()
        {
            System.out.println("after log ....");
        }
    }
  ```

  ```java
    //Main.java
    public class Main {
        public static void main(String[] args) {
          Car car  = new Car();
          car.run(120);
        }
    }
  ```
  运行结果
  ```text
    before log ....
    car is running at 120km/h
    after log ....
  ```

* aspectj编译后的代码，看aop时如何实现的

  ```java
    //Car.class
    public class Car {
        public Car() {
        }
        public void run(int speed) {
            try {
                AspectLog.aspectOf().beforeLog();
                System.out.println(String.format("car is running at %dkm/h", speed));
            } catch (Throwable var3) {
                AspectLog.aspectOf().afterLog();
                throw var3;
            }
            AspectLog.aspectOf().afterLog();
        }
    }
  ```

  ```java
    //Main.class
    public class Main {
        public Main() {
        }
        public static void main(String[] args) {
            Car car = new Car();
            car.run(120);
        }
    }
  ```

  ```java
    //traceLog.class
    import org.aspectj.lang.NoAspectBoundException;
    import org.aspectj.lang.annotation.After;
    import org.aspectj.lang.annotation.Aspect;
    import org.aspectj.lang.annotation.Before;

    @Aspect
    public class AspectLog {
        public AspectLog() {
        }

        @Before("logPointCut()")
        public void beforeLog() {
            System.out.println("before log ....");
        }

        @After("logPointCut()")
        public void afterLog() {
            System.out.println("after log ....");
        }

        public static AspectLog aspectOf() {
            if (ajc$perSingletonInstance == null) {
                throw new NoAspectBoundException("com.clutch.AspectLog", ajc$initFailureCause);
            } else {
                return ajc$perSingletonInstance;
            }
        }

        public static boolean hasAspect() {
            return ajc$perSingletonInstance != null;
        }

        static {
            try {
                ajc$postClinit();
            } catch (Throwable var1) {
                ajc$initFailureCause = var1;
            }
        }
    }
  ```

由以上编译后的代码可看出，aspectj在编译时已经将对应的代码织入到方法中

如果将AspectLog中pointcut的execution修改为call

  ```java
    @Aspect
    public class AspectLog {
        //将execution修改为call
        @Pointcut("call(* Car.run(..))")
        private void logPointCut(){}

        ...其他代码未变
    }
  ```
编译生成的代码是不同的

  ```java
    //Car.class
    public class Car {
        public Car() {
        }
        public void run(int speed) {
            System.out.println(String.format("car is running at %dkm/h", speed));
        }
    }
  ```
  ```java
    //Main.class
    public class Main {
        public Main() {
        }

        public static void main(String[] args) {
            Car car = new Car();
            Car var10000 = car;
            byte var10001 = 120;
            try {
                AspectLog.aspectOf().beforeLog();
                var10000.run(var10001);
            } catch (Throwable var3) {
                AspectLog.aspectOf().afterLog();
                throw var3;
            }
            AspectLog.aspectOf().afterLog();
        }
    }
  ```

对比后可以看出，execution是直接织入pointcut方法，而call则是织入调用pointcut方法的方法中。如果pointcut方法被众多其他方法调用时，且方法可被工程编译时，则使用execution更合适。
call适用于调用系统，或外部引用类库等方法等不可被工程编译的。

## PointCut

* 在方法上使用注解

  ```java
    //定义注解
    import java.lang.annotation.ElementType;
    import java.lang.annotation.Target;

    @Target({ElementType.TYPE, ElementType.METHOD})
    public @interface AspectTraceLog {
    }
  ```

  ```java
    //在方法加上注解
    @AspectTraceLog
    public void run(int speed) {
        System.out.println(String.format("car is running at %dkm/h", speed));
    }
  ```

  ```java
    @Aspect
    public class AspectLog {
        //修改pointcut的表达式
        @Pointcut("execution(@AspectTraceLog * *(..))")
        private void logPointCut(){ }

        ...其他代码未变
    }
  ```
* 在类上使用注解

  ```java
    //在类上加注释
    @AspectTraceLog
    public class Car {
        public void run(int speed) {
            System.out.println(String.format("car is running at %dkm/h", speed));
        }
        public void stop() {
            System.out.println("car has been stopped.");
            this.turnoff();
        }
        private void turnoff(){
            System.out.println("car turn off....");
        }
    }
  ```

  ```java
    @Aspect
    public class AspectLog {
        //使用组合表达式
        @Pointcut("within(@AspectTraceLog *) && execution(* *(..))")
        private void logPointCut(){ }

        ...其他代码未变
    }
  ```
  编译后的代码，可以看出私有方法也被织入了

    ```java
      @AspectTraceLog
      public class Car {
          public Car() {
          }

          public void run(int speed) {
              try {
                  AspectLog.aspectOf().beforeLog();
                  System.out.println(String.format("car is running at %dkm/h", speed));
              } catch (Throwable var3) {
                  AspectLog.aspectOf().afterLog();
                  throw var3;
              }
              AspectLog.aspectOf().afterLog();
          }

          public void stop() {
              try {
                  AspectLog.aspectOf().beforeLog();
                  System.out.println("car has been stopped.");
                  this.turnoff();
              } catch (Throwable var2) {
                  AspectLog.aspectOf().afterLog();
                  throw var2;
              }
              AspectLog.aspectOf().afterLog();
          }

          private void turnoff() {
              try {
                  AspectLog.aspectOf().beforeLog();
                  System.out.println("car turn off....");
              } catch (Throwable var2) {
                  AspectLog.aspectOf().afterLog();
                  throw var2;
              }
              AspectLog.aspectOf().afterLog();
          }
      }
    ```

* 同时兼容类和方法上的注解。先定义两个pointcut，分别为类和方法，然后在advice表达式上使用或关系适配pointcut
  ```java
    import org.aspectj.lang.ProceedingJoinPoint;
    import org.aspectj.lang.annotation.*;

    @Aspect
    public class AspectLog {

        @Pointcut("within(@AspectTraceLog *) && execution(* *(..))")
        private void classLogPointCut(){ }

        @Pointcut("execution(@AspectTraceLog * *(..))")
        private void methodLogPointCut(){ }

        @Before("classLogPointCut() || methodLogPointCut()")
        public void beforeLog()
        {
            System.out.println("before log ....");
        }

        @After("classLogPointCut() || methodLogPointCut()")
        public void afterLog()
        {
            System.out.println("after log ....");
        }
    }
  ```

* 获取切入点方法的相关信息，如类名、方法名、参数名、参数值、方法耗时等

  ```java
    import org.aspectj.lang.JoinPoint;
    import org.aspectj.lang.annotation.*;
    import org.aspectj.lang.reflect.MethodSignature;

    @Aspect
    public class AspectLog {

        @Pointcut("within(@AspectTraceLog *) && execution(* *(..))")
        private void classLogPointCut() {
        }

        @Pointcut("execution(@AspectTraceLog * *(..))")
        private void methodLogPointCut() {
        }

        @Before("classLogPointCut() || methodLogPointCut()")
        public void beforeLog(JoinPoint jp) {
            System.out.println("before log ....");
            printMethodInfo(jp);
        }

        private void printMethodInfo(JoinPoint jp) {
            MethodSignature signature = (MethodSignature) jp.getSignature();

            System.out.println(String.format("class: %s", signature.getDeclaringTypeName()));
            System.out.println(String.format("Method: %s", signature.getMethod().getName()));

            Object[] paramValues = jp.getArgs();
            String[] parameterNames = signature.getParameterNames();
            Class[] parameterTypes = signature.getParameterTypes();

            int index = 0;
            for (String para : parameterNames) {
                System.out.println(
                        String.format("parameter name: %s; type: %s; value: %s",
                                para,
                                parameterTypes[index],
                                paramValues[index]));
                index++;
            }
        }        
    }

  ```
  运行结果
  ```text
    before log ....
    class: com.clutch.Car
    Method: run
    parameter name: speed; type: int; value: 120
    car is running at 120km/h
  ```

* Around advice相当于同时包含了Before和After，以上代码就可以再简化下
  
  ```java
    @Around("classLogPointCut() || methodLogPointCut()")
    public void aroundLog(ProceedingJoinPoint jp) throws Throwable {
        Stopwatch sw = Stopwatch.createStarted();

        try {
            System.out.println("before log ....");
            printMethodInfo(jp);

            //调用pointcut方法
            jp.proceed();
        } catch (Throwable ex) {
            System.out.println("after log when throw exception....");
            throw ex;
        }

        sw.stop();
        System.out.println(String.format("method cost time:%d ms", sw.elapsed(TimeUnit.MILLISECONDS)));
        System.out.println("after log ....");
    }    
  ```

  运行结果
  ```text
    before log ....
    class: com.clutch.Car
    Method: run
    parameter name: speed; type: int; value: 120
    car is running at 120km/h
    method cost time:10 ms
    after log ....
  ```  