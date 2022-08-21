# 一、总结
- 当有volatile关键字修饰时，一定保证可见性
- 没有volatile关键字修饰时，不一定保证可见性，因为修改后何时写入main内存是不确定的
- volatile关键字并不能解决原子性问题
# 二、案列

```java public class AE implements Runnable{
程序期望出现的结果是打印：成功

public class AE implements Runnable{

public boolean sign=false;
@Override
public void run() {
    while (!sign){

    }
    System.out.println("成功");

}
}

public class ApiTest { public static void main(String[] args) {

    final AE ae=new AE();

    Thread thread01=new Thread(ae);
    Thread thread02=new Thread(new Runnable() {
        @Override
        public void run() {

            try {
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            ae.sign=true;
            System.out.println("通知 while(!sign)成功");
        }
    });

    thread01.start();
    thread02.start();
```

实际运行代码的时候，程序并没有出现期望值。而是处于死循环中。

加上volatile关键字之后，程序正常打印：成功

```java
public class AE implements Runnable{

    public volatile boolean sign=false;
    @Override
    public void run() {
        while (!sign){

        }
        System.out.println("成功");

    }
}
```

volatile关键字是jvm提供的最轻量级的同步机制。用来修饰变量（不包括局部变量），用来保证变量对所有线程的 **可见性**。当一个线程修改了变量，另一个线程会立即获取并作出相应反应。

# 二、可见性原理分析

## 2.1无volatile时，内存变化

![image.png](https://blog.aisummer.cn/upload/2021/10/image-96ca304bd13b488296252b608501c1c2.png)![](https://secure.wostatic.cn/static/pjHJcYG6SyFL2Qs2nGpcTU/image.png)



当sign没有关键字volatile修饰时，当线程1改变sign=true时，线程2并没有获取到sign的最新值。所以程序就不会出现预期的结果

## 2.2当有volatile关键字修饰时，内存变化

![image.png](https://blog.aisummer.cn/upload/2021/10/image-91ba3c9a570941bcb9f3c0e29254dd0d.png)![](https://secure.wostatic.cn/static/swpvCfuSSjeMUocbcumR31/image.png)

当变量加上volatile关键字后，pubilc volatile boolean sign=false; 线程1对sign进行修改后，会把变化后的值强制刷新到main 内存中，当线程2获取值时，会将自己内存中的sign=false；强制过期掉，之后从主内存中获取最新值，就会输出期望结果。

