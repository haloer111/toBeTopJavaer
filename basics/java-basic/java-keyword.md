### 1.transient
我们都知道一个对象只要实现了Serilizable接口，这个对象就可以被序列化，java的这种序列化模式为开发者提供了很多便利，我们可以不必关系具体序列化的过程，只要这个类实现了Serilizable接口，这个类的所有属性和方法都会自动序列化。

然而在实际开发过程中，我们常常会遇到这样的问题，这个类的有些属性需要序列化，而其他属性不需要被序列化，打个比方，如果一个用户有一些敏感信息（如密码，银行卡号等），为了安全起见，不希望在网络操作（主要涉及到序列化操作，本地序列化缓存也适用）中被传输，这些信息对应的变量就可以加上transient关键字。换句话说，这个字段的生命周期仅存于调用者的内存中而不会写到磁盘里持久化。

总之，java 的transient关键字为我们提供了便利，你只需要实现Serilizable接口，将不需要序列化的属性前添加关键字transient，序列化对象的时候，这个属性就不会序列化到指定的目的地中。

#### 小结：

1. 一旦变量被transient修饰，变量将不再是对象持久化的一部分，该变量内容在序列化后无法获得访问。

2. transient关键字只能修饰变量，而不能修饰方法和类。注意，本地变量是不能被transient关键字修饰的。变量如果是用户自定义类变量，则该类需要实现Serializable接口。

3. 被transient关键字修饰的变量不再能被序列化，一个静态变量不管是否被transient修饰，均不能被序列化

### 2.instanceof

instanceof 严格来说是Java中的一个双目运算符，用来测试一个对象是否为一个类的实例，用法为：
```
boolean result = obj instanceof Class
```

1、obj 必须为引用类型，不能是基本类型
```
int i = 0;
System.out.println(i instanceof Integer);//编译不通过
System.out.println(i instanceof Object);//编译不通过
```

2、obj 为 null
```
System.out.println(null instanceof Object);//false
```

3、obj 为 class 类的实例对象
```
Integer integer = new Integer(1);
System.out.println(integer instanceof  Integer);//true
```

4、obj 为 class 类的直接或间接子类
```
# 我们新建一个父类 Person.class，然后在创建它的一个子类 Man.class
public class Person {
}

public class Man extends Person{
}

# 测试：
Person p1 = new Person();
Person p2 = new Man();
Man m1 = new Man();
System.out.println(p1 instanceof Man);//false
System.out.println(p2 instanceof Man);//true
System.out.println(m1 instanceof Man);//true
```

### 3.volatile

`volatile`关键字的主要作用是使变量在多个线程间可见，但是却不具备同步性（也就是原子性），可以算上是一个轻量级的`synchronized`，性能要比`synchronized`强很多，不会造成阻塞（在很多开源的架构里，比如netty的底层代码就大量使用`Volatile`）这里需要注意，一般`volatile`用于只针对于多个线程可见的变量操作，并不能代替`synchronized`的同步功能。

下面看一个例子：
```
    public class RunThread implements Runnable {
        private boolean isRunning = true;
     
        public boolean isRunning() {
            return isRunning;
        }
     
        public void setRunning(boolean isRunning) {
            this.isRunning = isRunning;
        }
     
        @Override
        public void run() {
            System.out.println("进入run了");
            while (isRunning) {
     
            }
            System.out.println("线程被停止了！");
        }
     
        public static void main(String[] args) throws InterruptedException {
            RunThread thread = new RunThread();
            new Thread(thread).start();
            Thread.sleep(100);
            thread.setRunning(false);
            System.out.println("已经赋值为false");
        }
    }
```
结果：

    进入run了
    已经赋值为false

实例中的线程有一个通过判断isRunning的真假实现的循环，默认是true，我们在测试方法中将isRunning改为了false，但是根据结果显示循环仍然在继续，也就是我们赋的值并没有起作用。 

现在在isRunning的声明前面加上volatile关键字再试试：

volatile private boolean isRunning = true;

结果：

    进入run了
    已经赋值为false
    线程被停止了！

可以看到这次循环就被停止了，我们的赋值操作起作用了，volatile到底怎么实现的呢？ 
在启动RunThread.java线程时，变量isRunning存在于公共堆栈和线程的私有堆栈两个地方，为了提升效率，线程一直在默认堆栈中取值，所以取得的isRunning一直是true；而代码thread.setRunning(false);更新的却是公共堆栈中isRunning的值，因此这就造成了公共堆栈中isRunning的值是false而线程持有的私有堆栈中isRunning的值却是true，所以没有volatile关键字时线程是死循环状态。 
这样的问题其实就是私有堆栈中的值和公共堆栈中的值不同步造成的，解决这样的问题就要用volatile关键字了，它主要的作用就是当线程访问isRunning这个变量时，强制性从公共堆栈中取值。 

volatile关键字只具有可见性，没有原子性。要实现原子性建议使用atomic类的系列对象，支持原子性操作（注意atomic类只保证本身方法原子性，并不保证多次操作的原子性）。