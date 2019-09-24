# JAVA黑马教程

## 多线程

线程安全问题产生原因：

* 共享数据
* 超过2行代码

---

**wait和sleep的区别：**

* wait可以制定时间也可以不指定，sleep必须指定时间
* 在同步中，对cpu的执行权和锁的处理不同，wait：释放执行权，释放锁；sleep：释放执行权，不释放锁；

---

**停止线程:**  
1. stop方法，不安全，不推荐；  
2. run\(\)方法结束，通过设置标记  
3. 使用interrupt\(\)方法，将线程从冻结状态强制恢复至运行状态中来，让线程具备CPU执行资格，但是强制动作会发生中断异常，需要处理。

线程中的任务一般在循环中，只要能够控制住循环，就可以结束任务

控制循环的方法就只能通过标记完成

但是线程处于冻结状态就无法读取标记

---

### 线程安全

当一个线程通过多行代码操作共享数据，未执行完成时其他线程参与了运算，从而导致**线程安全问题**

**解决方案：**

synchronized\(obj\){代码块}

obj为同步锁

* 好处：解决线程安全 
* 弊端：性能下降，判断同步锁

**同步的前提：**

必须有多线程，并使用同一个锁。

_**三类锁：**_

* 自定义锁；
* 同步函数使用的锁是this锁；
* 静态同步方法使用，.getClass\(\)或字节码类名.class ，锁为该函数所属字节码文件对象

_**同步函数与同步代码块的区别：**_

* 同步函数的锁是固定的this；
* 同步代码块的锁是任意；
* 实际开发中，建议使用同步代码块。

### 单例设计模式

_**"饿汉模式"：**_

```java
class Single
{
    private static final Single s = new Single();
    private Single(){}
    public static Single getInstance(){
        return s;
    }
}
```

_**"懒汉模式":**_

```java
class Single
{
    private static Single s = null;
    private Single(){}
    public static Single getInstance()
    {
        if(s==null){
            synchronized(Single.class)
            {
                if(s==null)
                    s = new Single();
            }
        }
        return s;
    }
}
```

### 死锁

同步嵌套，交叉引用。才会造成死锁

### 线程间通信

_**等待唤醒机制：**_

* wait\(\);让线程处于冻结状态，释放CPU执行权与执行资格，wait的线程会被存储的线程池中。
* notify\(\);唤醒线程池中的一个线程（任意）
* notifyAll\(\);唤醒线程中的所有线程

这些方法必须定义再同步中，因为这些方法是用于操作进程状态的方法，必须明确操作的是哪个锁上的线程。

```java
class Resource {
    private String name;
    private String sex;
    private boolean flag = false;

    public synchronized void set(String name, String sex) {
        if (flag)
            try {
                this.wait();
            } catch (InterruptedException e) {
            }
        this.name = name;
        this.sex = sex;
        flag = true;
        this.notify();
    }

    public synchronized void out() {
        if (!flag)
            try {
                this.wait();
            } catch (InterruptedException e) {
            }
        System.out.println(this.name + "....." + this.sex);
        flag = false;
        this.notify();
    }
}

class Input implements Runnable {

    Resource r;

    Input(Resource r) {
        this.r = r;
    }

    public void run() {
        int x = 0;
        while (true) {
            if (x == 0) {
                r.set("麦克", "男");
            } else {
                r.set("丽丽", "女");
            }
            x = (x + 1) % 2;
        }
    }
}


class Output implements Runnable {
    Resource r;

    Output(Resource r) {
        this.r = r;
    }

    public void run() {

        while (true) {
            r.out();
        }
    }
}

public class SaleTicket {
    public static void main(String[] args) {
        Resource r = new Resource();
        Input in = new Input(r);
        Output out = new Output(r);

        Thread t1 = new Thread(in);
        Thread t2 = new Thread(out);

        t1.start();
        t2.start();
    }
}
```

_**生产者消费者模式**_

while判断标记，解决了线程获取执行权后，是否要运行业务逻辑

notifyAll\(\)方法，解决了本方线程一定唤醒对方线程，从而解决可能的死锁问题

```java
class Resource {
    private String name;
    private int count = 1;
    private boolean flag = false;

    public synchronized void set(String name) {

        while (flag)    //if --> while 解决线程唤醒后需要判断标记的问题，以保证线程安全
            /*
            但是仍然存在问题，会出现死锁问题，为解决该问题，唤醒使用notifyAll(){而不使用notify()}方法
             */
            try {
                this.wait();
            } catch (InterruptedException e) {
            }
        this.name = name + count;
        count++;

        System.out.println(Thread.currentThread().getName() + "...生产者...做" + this.name);
        this.flag = true;
        notifyAll();
    }

    public synchronized void out() {
        while (!flag)
            try {
                this.wait();
            } catch (InterruptedException e) {
            }
        System.out.println(Thread.currentThread().getName() + "...消费者.......吃" + this.name);
        flag = false;
        notifyAll();
    }
}


class Provider implements Runnable {

    private Resource r;

    Provider(Resource r) {
        this.r = r;
    }

    @Override
    public void run() {
        while (true) {
            r.set("烤鸭");
        }
    }
}

class Consumer implements Runnable {
    private Resource r;

    Consumer(Resource r) {
        this.r = r;
    }

    @Override
    public void run() {
        while (true) {
            r.out();
        }
    }
}


public class Demo {
    public static void main(String[] args) {
        Resource r = new Resource();
        Provider provider = new Provider(r);
        Consumer consumer = new Consumer(r);

        Thread t1 = new Thread(provider);
        Thread t2 = new Thread(consumer);
        Thread t3 = new Thread(provider);
        Thread t4 = new Thread(consumer);

        t1.start();
        t2.start();
        t3.start();
        t4.start();

    }
}
```

---

_jdk升级后有更优的解决方案，因为notifyAll\(\)降低了效率，能否做到只唤醒对方线程呢？_

java.util.concurrent.locks

**接口**

_Lock_

使用同步代码块，对于锁的操作是隐式的；使用Lock接口是显示的。

Lock接口：替代了同步代码块或者同步函数，将同步的隐式锁操作变成了显式锁操作。

* lock\(\)：获取锁
* unlock\(\)：释放锁，需要在finally中

Condition接口：替代了监视器对象。可与lock自由组合。

```java
Lock lock = new ReentrantLock();    //自定义锁，互斥锁

/*
1.5以后将同步和锁封装成了对象。
并将操作锁的隐式方式定义到了该对象中，
将隐式方式换成显示方式
*/

void show(){
    try{
            lock.lock();
            code...
    }
    finally{
        lock.unlock()
    }
}
```

一个Lock带多组Condition示例代码

```java
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

class Resource {
    private String name;
    private int count = 1;
    private boolean flag = false;

    Lock lock = new ReentrantLock();
    /*1.5之后的Condition接口实现了，一个lock获得多组监视器，从而实现只唤醒对方进程的功能*/
    Condition conditionProvider = lock.newCondition();
    Condition conditionConsumer = lock.newCondition();

    public void set(String name) {
        lock.lock();
        try {
            while (flag)    //if --> while 解决线程唤醒后需要判断标记的问题，以保证线程安全
                /*
                但是仍然存在问题，会出现死锁问题，为解决该问题，唤醒使用notifyAll()方法
                 */
                try {
                    conditionProvider.await();
                } catch (InterruptedException e) {
                }

            this.name = name + count;
            count++;

            System.out.println(Thread.currentThread().getName() + "...生产者...做" + this.name);
            this.flag = true;
            conditionConsumer.signal();

        } finally {
            lock.unlock();
        }
    }

    public void out() {
        lock.lock();
        try {
            while (!flag)
                try {
                    conditionConsumer.await();
                } catch (InterruptedException e) {
                }
            System.out.println(Thread.currentThread().getName() + "...消费者.......吃" + this.name);
            flag = false;
            conditionProvider.signal();
        } finally {
            lock.unlock();
        }
    }
}


class Provider implements Runnable {

    private Resource r;

    Provider(Resource r) {
        this.r = r;
    }

    @Override
    public void run() {
        while (true) {
            r.set("烤鸭");
        }
    }
}

class Consumer implements Runnable {
    private Resource r;

    Consumer(Resource r) {
        this.r = r;
    }

    @Override
    public void run() {
        while (true) {
            r.out();
        }
    }
}


public class Demo {
    public static void main(String[] args) {
        Resource r = new Resource();
        Provider provider = new Provider(r);
        Consumer consumer = new Consumer(r);

        Thread t1 = new Thread(provider);
        Thread t2 = new Thread(consumer);
        Thread t3 = new Thread(provider);
        Thread t4 = new Thread(consumer);

        t1.start();
        t2.start();
        t3.start();
        t4.start();

    }
}
```

## JAVA API

### String类

**特点:**

字符串对象一旦被初始化就不会被改变

```java
/*
String s = "abc";//创建于字符串常量池
String s1 = new String("abc");//创建于堆内存中

s or s1是变量， 而"abc"是字符串对象，创建之后进入字符串
*/
package cn.liuz.string.demo;

public class Demo {
    public static void main(String[] args) {
        String string = "abc";
        String string1 = new String("abc");
        System.out.println(string == string1);
        System.out.println(string.equals(string1));
    }
}
```

**方法：**

1.获取：

1.1 获取字符串的长度

`int length();`

1.2 根据位置获取字符

`char charAt(int index);`

1.3 根据字符获取在字符串中第一次出现的位置

`int indexOf(int ch);`

`int indexOf(int ch, int fromIndex);`

`int indexOf(String str, int fromIndex);`





