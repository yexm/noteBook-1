# JAVA黑马教程

## 1 多线程

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

### 1.1线程安全

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

### 1.2单例设计模式

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

### 1.3死锁

同步嵌套，交叉引用。才会造成死锁

### 1.4线程间通信

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

## 2 JAVA API

### 2.1 String

#### 2.1.1 String类

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

```java
int length();
```

1.2 根据位置获取字符

```java
char charAt(int index);
```

1.3 根据字符获取在字符串中第一次出现的位置

```java
int indexOf(int ch);
int indexOf(int ch, int fromIndex);
int indexOf(String str, int fromIndex);
```

1.4 获取子串

```java
String substring(int begin, int end)

//不包含 end
```

2.转换：

2.1 将字符串切片

```java
String[] split(String regex)
```

2.2 将字符串、字节返回字符、字节数组

```java
char[] toCharArray();
byte[] getBytes();
```

2.3 大小写转换

```java
String toUppercase();
String toLowercase();
```

2.4 字符串修改

```java
String replace(Char oldChar, Char newChar)
String replace(String s1, String s2)
```

2.5 清理字符串前后空格

```java
String trim()
```

2.6 字符串连接

```java
String concat(String string)
```

3.判断

3.1 两个字符串是否相同

```java
boolean equals(Object obj);
boolean equalsIgnoreCase(Object obj);
```

3.2 字符串包含

```java
boolean contains(String string)
```

3.3 以指定字符串开头/结尾

```java
boolean startsWith(String str);
boolean endsWith(String str);
```

4.比较

4.1比较字符串的大小

```java
int compareTo(String anotherString)
```

#### 2.1.2 StringBuffer类

字符串缓冲区，用于临时存储数据

特点：

1. 长度可变
2. 可以存储不同数据类型
3. 最终转成字符串使用

是一个容器对象，具备以下功能CURD操作：

添加：

```java
StringBuffer append(data)
StringBuffer insert(index, data)
```

删除：

```java
StringBuffer delete(start, end) //含头不含尾(0,str.lenght())清空缓冲区
StringBuffer deleteCharAt(int index) //删除指定位置的元素

//注意setLength(int length)方法，length=0相当于清空字符串
```

查找, 同字符串

修改：

```java
StringBuffer replace(start, end, string)
void setCharAt(int index, char chr)
```

#### 2.1.3 StringBuilder类\(1.5版本后产生\)

功能和用法一模一样，StringBuilder不保证线程安全

StringBuilder用于单线程。

### 2.2 基本数据类型对象包装类

基本数据类型：

byte --&gt; Byte  
short --&gt; Short  
int --&gt; Integer  
long --&gt; Long  
float --&gt; Float  
double --&gt; Double  
char --&gt; Character  
boolean --&gt; Boolean

### 2.3 集合框架

对象用于封装特有数据，多了就需要存储，对象个数不确定就使用集合容器；

**特点：**

1. 用于存储对象的容器；
2. 集合的长度可变；
3. 集合中不可以存储基本数据类型值。

框架的顶层Collection接口：

1.添加:

```java
boolean add(Object obj);
boolean addAll(Collection coll);
```

2.删除:

```java
boolean remove(Object obj);
boolean removeAll(Collection coll);
void clear();
```

3.判断：

```java
boolean contains(Object obj);
boolean containsAll(Collection coll);
boolean isEmpty();
```

4.获取：

```java
int size();
Iterator iterator();
```

5.其他：

```java
boolean retainALL(Collection coll);
Object[] toArray();
```

```java
Collection
    | -- List, 有序，可重复
    | -- Set, 无序，不可重复·
```

#### List特有的常见方法

共性特点，可以操作角标

1. 添加：
   void add\(index, element\);
   void addAll\(index, collection\);
2. 删除：
   Object remve\(index\);
3. 修改：
   Object set\(index, element\);
4. 获取：
   Object get\(index\);
   int IndexOf\(object\);
   int lastIndexOf\(object\);
5. 切片：
   List subList\(start, end\);

listIterator\(\)返回ListIterator

```java
List:
    |--Vector: 内部数据结构是数组。每个元素有编号，连续存储。线程安全。增删、查询够很慢。
    |--ArrayList:内部数组数据结构。不同步。使用单线程，效率高。Vector几乎不用了。查询效率高。
    |--LinkedList:内部是连表数据结构。不同步，效率高。增删效率高。
    
    
LinkList
在JDK1.6后
getFirst(); --> peekFirst();
getLast(); --> peekLask();

removeFirst(); --> pollFirst();
removeLast(); --> pollLast();
```

#### Set常见方法

Set接口中的方法与Conllection方法一致

```java
    |--HashSet：内部数据结构是哈希表，线程不安全；
        通过hashCode和equals方法来完成对象的唯一性，
        if hashCode不相同，直接存储到hash表中；
        else 使用equals方法判断
            if true
                视为相同元素不存
            else
                进行存储
        *** 首先计算hashcode，相同再使用equals方法判定
        注意：使用HashSet一定要覆写hashCode和equals方法
    |--TreeSet：自然顺序排序，可以对Set集合中的元素进行排序
                判断元素唯一性的方式：实现comparable接口，覆盖compareTo()
                如果不要按照对象中具备的自然顺序，若果对象不具备自然顺序，可以使用TreeSet集合的第二种方式：
                让集合具备比较功能，定义类实现Comparator接口，覆盖compare方法，将该类对象作为参数传递给TreeSet集合的构造函数，比较器相对常用一些。
```

#### 泛型

1.5出现的，安全机制

**好处：**

1. 将运行时期的问题转ClassCastException到了编译时期；
2. 避免了强制转换的麻烦。

<>当操作引用数据类型，不确定的时候，就使用泛型，将要操作的引用数据类型传入即可；就是用于接收具体引用数据类型参数范围。

在程序中只要用到了带有<>的类或者接口，就要明确传入具体的引用数据类型。为了兼容类加载器。

泛型用于 ***编译时期***，确保类型的安全
运行时会将泛型去掉，生成的class文件中，是不带泛型的，这称之为泛型的擦除。
泛型的补偿，运行时通过获取元素类型进行转换动作，不用使用者强制转换了。

```java
/**
 * Copyright (C), 2018-2019, Personality
 * FileName: GD2
 * Author:   liuz
 * Date:     2019/10/17 17:28
 * Description:
 * History:
 * <author>          <time>          <version>          <desc>
 * 作者姓名           修改时间           版本号              描述
 */
package com.liuz.generic.demo;

import java.util.Comparator;
import java.util.Iterator;
import java.util.Objects;
import java.util.TreeSet;

/**
 * 〈一句话功能简述〉<br>
 * 〈〉
 *
 * @author liuz
 * @create 2019/10/17
 * @since 1.0.0
 */

class Person implements Comparable<Person> {
    private String name;
    private Integer age;

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Person person = (Person) o;
        return Objects.equals(name, person.name) &&
                Objects.equals(age, person.age);
    }

    @Override
    public int hashCode() {
        return Objects.hash(name, age);
    }

    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }

    @Override
    public int compareTo(Person o) {
        return this.name.compareTo(o.name) == 0 ? this.age - o.age : this.name.compareTo(o.name);
    }

    public Person() {
        super();
    }

    public Person(String name, Integer age) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }
}


class OrderByAge implements Comparator<Person> {
    @Override
    public int compare(Person o1, Person o2) {
        return o1.getAge() - o2.getAge() == 0 ? o1.getName().compareTo(o2.getName()) : o1.getAge() - o2.getAge();
    }
}


public class GD2 {
    public static void main(String[] args) {
        TreeSet<Person> ts = new TreeSet<Person>(new OrderByAge());

        ts.add(new Person("zhangsan", 21));
        ts.add(new Person("lisi", 22));
        ts.add(new Person("wangwu", 21));
        ts.add(new Person("zhaoliu", 25));

        Iterator<Person> it = ts.iterator();

        while (it.hasNext()) {
            System.out.println(it.next());
        }
    }

}
```

泛型的自定义演示

泛型类的使用，在于不确定传入的引用类型

```java
public class Tools<E> {

    private E object;

    public E getObject() {
        return object;
    }

    public void setObject(E object) {
        this.object = object;
    }
    // 泛型定义再方法上接收任意类型参数
    public <W> void show(W obj) {
        System.out.println("show: " + obj.toString());
    }

}
```

当方法静态时，不能访问类上定义的泛型，如果使用泛型，只能将泛型定义在方法上。
还可以使用泛型接口，具体深刻时候指定类型，随意。
泛型通配符：?未知类型

泛型限定，可以对类型限定：

```bash
<? extends E>接收E类型或E的子类型对象，上限！
#一般在存储元素的时候使用上限。不会出现类型安全隐患。

<? super E>接收E类型或E类型父类型对象，下限！
#一般在去除元素的时候使用下限。
```

#### 集合使用技巧

**需要唯一Set**
需要顺序TreeSet；不需要HashSet，但是想要一个和存储一致的顺序：LinkedHashSet

**不需要唯一List**
需要频繁增删LinkedList，不需要ArrayList

```bash
看到array，数组，查询快，有角标；
看到link，链表，增删快，有add,get,remove+first,last方法
看到hash,哈希表，覆盖hashcode和equals
看到tree，二叉树，需要实现Comparable或Comparator接口

以上均为不同步
```

#### Map集合

接口 Map<K,V>

一次添加一对元素，Conllection一次添加一个元素，Map称为双列结合。

Map集合中必须保证key的唯一性，与python的字典类型一致。

##### 常用方法：

1.添加
value put(key,value):return前一个和key关联的值，如果没有返回null
2.删除
void clear();清空Map集合
value remove(key):根据指定的key删除键值对
3.判断
boolean containsKey(key);
boolean containsValue(value);
boolean isEmpty();
4.获取
value get(key);可以通过返回空判断是否包含指定键
int size();


##### 常用子类

```
	|-- Hashtable: 内部结构是哈希表，是同步的；不支持null key或null value。
		|-- Properties：用来存储键值对型的配置文件信息，可以和IO技术结合。
	|-- HashMap：内部结构哈希表，不同步；允许null key/value
	|-- TreeMap: 二叉树，不同步；可以对Map中的key排序。
```

#### 工具类

Collections集合框架工具类，里面的方法都是静态的。