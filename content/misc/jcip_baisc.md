---
title: "Java并发编程实战::基础知识"
date: 2021-08-23T15:55:20+08:00
draft: true
---

## 线程安全性

线程安全性的定义：当多个线程访问某个类时，这个类始终都能表现出正确的行为，那么就称这个类时线程安全的。在线程安全性的定义中，最核心的概念就是正确性。

### 原子性

在并发编程中，这种由于不恰当的执行时序而出现不正确的结果是一种非常重要的情况，它有一个正式的名字：竞态条件(Race Condition).

数据争用，线程分析器可检测多线程进程执行期间发生的数据争用。满足以下所有条件时，就会发生数据争用：一个进程内的两个或多个线程同时访问同一内存位置 ，至少其中一个访问是用于写入 ，线程未使用任何互斥锁来控制它们对该内存的访问

### 加锁机制

#### 内置锁

Java 提供了一种内置的锁机制来支持原子性：同步代码块 (Synchronized Block)。同步代码块包括两部分： 一个作为锁的对象引用， 一个作为由这个锁保护的代码块。以关键字 synchronized 来修饰的方法就是一种横跨整个方法体的同步代码块，其中该同步代码块的锁就是方法调用所在的对象。静态的 synchronized 方法以 Class 对象作为锁。

```java
synchronized(lock){
 // 访问或修改由锁保护的共享状态
}
```

每个 Java 对象都可以用做一个实现同步的锁，这些锁被称为内置锁 (Intrinsic Lock) 或监视器锁 (Monitor Lock)。线程在进入同步代码块之前会自动获得锁，并且在退出同步代码块时自动释放锁，而无论是通过正常的控制路径退出，还是通过从代码块中抛出昇常退出。获得内置锁的唯一途径就是进入由这个锁保护的同步代码块或方法。

Java 的内置锁相当于一种互斥体（或互斥锁），这意味着最多只有一个线程能持有这种锁。当线程 A 尝试获取一个由线程 B 持有的锁时，线程 A 必须等待或者阻塞，直到线程 B 释放这个锁。如果 B 永远不释放锁，那么 A 也将永远地等下去。

由于每次只能有一个线程执行内置锁保护的代码块，因此，由这个锁保护的同步代码块会以原子方式执行，多个线程在执行该代码块时也不会相互干扰。并发环境中的原子性与事务应用程序中的原子性有着相同的含义，一组语句作为一个不可分割的单元被执行。任何一个执行同步代码块的线程，都不可能看到有其他线程正在执行由同一个锁保护的同步代码块。

#### 可重入

当某个线程请求一个由其他线程持有的锁时，发出请求的线程就会阻塞。然而，由于内置锁是可重入的，因此如果某个线程试图获得一个已经由它自己持有的锁，那么这个请求就会成功。 “重入”意味着获取锁的操作的粒度是“线程”，而不是“调用”。重入的一种实现方法是，为每个锁关联一个获取计数值和一个所有者线程。当计数值为口时，这个锁就被认为是没有被任何线程持有。当线程请求一个木被持有的锁时，JVM 将记下锁的持有者，并且将获取计数值置为 1。如果同一个线程再次获取这个锁，计数值将递增，而当线程退出同步代码块时，计数器会相应地递减。当计数值为 0 时，这个锁将被释放。

### 用锁来保护状态

访问共享状态的复合操作，例如命中计数器的递增操作（读取一修改一写入）或者延迟初始化（先检查后执行），都必须是原子操作以避免产生竞态条件。如果在复合操作的执行过程中持有一个锁，那么会使复合操作成为原子操作。然而，仅仅将复合操作封装到一个同步代码块中是不够的。如果用同步来协调对某个变量的访问，那么在访问这个变量的所有位置上都需要使用同步。而且，当使用锁来协调对某个变量的访问时，在访问变量的所有位置上都要使用同一个锁

对于可能被多个线程同时访问的可交状态变量，在访问它时都需要特看同一个锁，在这种情况下；我们称状态变量是由这个锁保护的

虽然 synchronized 方法可以确保单个操作的原子性，但如果要把多个操作合并为一个复合操作，还是需要额外的加锁机制

### 活跃性与性能

当使用锁时，你应该请楚代码块中实现的功能，以及在执行该代码块时是否需要很长的时间。无论是执行计算密集的操作，还是在执行某个可能阻塞的操作，如果持有锁的时间过长，那么都会带来活跃性或性能问题

## 对象的共享

我们已经知道了同步代码块和同步方法可以确保以原子的方式执行操作，但一种常见的误解是，认为关键字 synchronized 只能用于实现原子性或者确定“临界区 (Ctitical Section)”。同步还有另一个重要的方面：内存可见性 (Memory Visibility)。我们不仅希望防止某个线程正在使用对象状态而另一个线程在同时修改该状态，而且希望确保当一个线程修改了对象状态后其他线程能够看到发生的状态变化。如果没有同步，那么这种情况就无法实现。你可以通过显式的同步或者类库中内置的同步来保证对象被安全地发布。

### 可见性

可见性是一种复杂的属性，因为可见性中的错误总是会违背我们的直觉。在单线程环境中，如果向某个变量先写入值，然后在没有其他写入操作的情况下读取这个变量，那么总能得到相同的值。这看起来很自然。然而，当读操作和写操作在不同的线程中执行时，情况却并非如此，这听起来或许有些难以接受。通常，我们无法确保执行读操作的线程能适时地看到其他线程写入的值，有时甚至是根本不可能的事情。为了确保多个线程之间对内存写入操作的可见性，必须使用同步机制。

在没有同步的情况下，编译器、处理器以及运行时等都可能对操作的执行顺序进行一些意想不到的调整。在缺乏足够同步的多线程程序中，要想对内存操作的执行顺序进行判断，几乎无法得出正确的结论。

#### 失效数据

当读线程查看变量时，可能会得到一个已经失效的值。除非在每次访问变量时都使用同步，否则很可能获得该变量的一个失效值。更糟糕的是，失效值可能不会同时出现：一个线程可能获得某个变量的最新值，而获得另一个变量的失效值。

#### 非原子的 64 位操作

当线程在没有同步的情况下读取变量时，可能会得到一个失效值，但至少这个值是由之前某个线程设置的值，而不是一个随机值。这种安全性保证也被称为最低安全性(out-of-thin-air safety).

最低安全性适用于绝大多数变量，但是存在一个例外：非 volatile 类型的 64 位数值娈量。Java 内存模型要求，变量的读取操作和写入操作都必须是原子操作，但对于非 volatile 类型的 long 和 double 变量，JVM 允许将 64 位的读操作或写操作分解为两个 32 位的操作。当读取一个非 volatile 类型的 long 变量时，如果对该变量的读操作和写操作在不同的线程中执行，那么很可能会读取到某个值的高 32 位和另一个值的低 32 位。因此，即使不考虑失效数据问题，在多线程程序中使用共享且可变的 long 和 double 等类型的量也是不安全的，除非用关键宇 volatile 来声明它们，或者用锁保护起来。

#### 加锁与可见性

加锁的含义不仅仅局限于互斥行为，还包括内存可见性。为了确保所有线程都能看到共享变量的最新值，所有执行读操作或写操作的线程都必须在统一个锁上同步。

#### Volatile 变量

Java 语言提供了一种稍弱的同步机制，即 volatile 变量，用来确保将变量的更新操作通知到其他线程。当把变量声明为 volatile 类型后，编译器与运行时都会注意到这个变量是共享的，因此不会将该变量上的操作与其他内存操作一起重排序。volatile 变量不会被缓存在奇存器或者对其他处理器不可见的地方，因此在读取 volatile 类型的变量时总会返回最新写入的值。

volatile 的语义不足以确保递增操作 count++ 的原子性，除非你能确保只有一个线程对变量执行写操作。

当且仅当满足以下所有条件时，才应该使用 volatile 变量： 对变量的写入操作不依赖变量的当前值，或者你能确保只有单个线程更新变量的值，该变量不会与其他状态变量一起纳入不变性条件中，在访问变量时不需要加锁

加锁机制既可以确保可见性又可以确保原子性，而 volatile 变量只能确保可见性

#### 发布与逸出

“发布（Publish）“一个对象的意思是指，使对象能够在当前作用域之外的代码中使用。例如，将一个指向该对象的引用保存到其他代码可以访问的地方，或者在某一个非私有的方法中返回该引用，或者將引用传递到其他类的方法中。在许多情况中，我们要确保对象及其内部状态不被发布。而在某些情况下，我们又需要发布某个对象，但如果在发布时要确保线程安全性，则可能需要同步。发布内部状态可能会破坏封装性，并使得程序难以维持不变性条件。例如，如果在对象构造完成之前就发布该对象，就会破坏线程安全性。当某个不应该发布的对象被发布时，这种情况就被称为逸出(Escape)。

当发布一个对象时，在该对象的非私有域中引用的所有对象同样会被发布。一般来说，如果一个已经发布的对象能够通过非私有的变量引用和方法调用到达其他的对象，那么这些对象也都会被发布。

不要在构造过程中使 this 引用逸出。

### 线程封闭

当访问共享的可变数据时，通常需要使用同步。一种避免使用同步的方式就是不共享数据。如果仅在单线程内访问数据，就不需要同步。这种技术被称为线程封闭，它是实现线程安全性的最简单方式之一。当某个对象封闭在一个线程中时，这种用法将自动实现线程安全性，即使被封闭的对象本身不是线程安全的

在 Java 语言中并没有强制规定某个变量必须由锁来保护，同样在 Java 语言中也无法强制将对象封闭在某个线程中。线程封闭是在程序设计中的一个考虑因素，必须在程序中实现 Java 语言及其核心库提供了一些机制来帮助维持线程封闭性，例如局部变量和 ThreadLocal 类但即便如此，程序员仍然需要负责确保封闭在线程中的对象不会从线程中逸出

#### Ad-hoc 线程封闭

Ad-hoc 线程封闭是指，维护线程封闭性的职责完全由程序实现来承担。Ad-hoc 线程封闭是非常脆弱的，因为没有任何一种语言特性，例如可见性修饰符或局部变量，能将对象封闭到目标线程上。事实上，对线程封闭对象（例如，GUI 应用程序中的可视化组件或数据模型等）的引用通常保存在公有变量中

#### 栈封闭

栈封闭是线程封闭的一种特例，在栈封闭中，只能通过局部变量才能访问对象。正如封装能使得代码更容易维持不变性条件那样，同步变量也能使对象更易于封闭在线程中。局部变量的固有属性之一就是封闭在执行线程中。它们位于执行线程的栈中，其他线程无法访问这个栈。栈封闭比 Ad-hoc 线程封闭更易于维护，世更加健壯。

#### ThreadLocal

维持线程封闭性的一种更规范方法是使用 Threadlocal，这个类能使线程中的某个值与保存值的对象关联起来。ThreadLocal 提供了 get 与 set 等访问接口或方法，这些方法为每个使用该变量的线程都存有一份独立的副本，因此 get 总是返回由当前执行线程在调用 set 时设置的最新值．

ThreadLocal 对象通常用于防止对可变的单实例变量（Singleton）或全局变量进行共享

当某个线程初次调用 ThreadLocal.get 方法时，就会调用 initialValue 来获取初始值。从概念上看，你可以将 `ThreadLocal<T>`视为包含了`Map<Thread,T>`对象，其中保存了特定于该线程的值，但`ThreadLocal`的实现并非如此。这些特定于线程的值保存在`Thread`对象中，当线程终止后，这些值会作为垃圾回收。

假设你需要将一个单线程应用程序移植到多线程环境中，通过将共享的全局变量转換为 ThreadLocal 对象，可以维持线程安全性。然而，如果将应用程序范围内的缓存转换为线程局部的缓存，就不会有太大作用。

#### 不变性

如果某个对象在被创建后其状态就不能被修改，那么这个对象就称为不可变对象。线程安全性是不可变对象的固有属性之一，它们的不变性条件是由构造函数创建的，只要它们的状态不改变，那么这些不变性条件就能得以维持

不可变对象一定是线程安全的。

当满足一下条件时，对象才是不可变的：对象创建以后其状态就不能改变，对象的所有域都是 final 类型，对象是正确创建的。

#### 安全发布

要安全地发布一个对象，对象的引用以及对象的状态必须同时对其他线程可见。一个正确构造的对象可以通过以下方式来安全地发布：

- 在静态初始化函数中初始化一个对象引用
- 将对象的引用保存到 volatile 类型的域或者 AtomicReferaace 对象中。
- 将对象的引用保存到某个正确构造对象的 final 类型域中。
- 将对象的引用保存到一个由锁保护的域中。

在没有额外的同步的情况下，任何线程都可以安全地使用被安全发布的事实不可变对象。

对象的发布需求取决于它的可变性：

- 不可变对象可以通过任意机制来发布
- 事实不可变对象必须通过安全方式来发布
- 可变对象必须通过安全方式来发布，而且必须是线程安全的或者由某个锁保护起来。

### 对象的组合

通过使用封装技术，可以使得在不对整个程序进行分析的情况下就可以判断一个类是否是线程安全的。

在设计线程安全类的过程中，需要包含一下三个基本要素：

- 找出构成对象状态的所有变量
- 找出约束状态变量的不可变条件
- 建立对象状态的并发访问管理策略

#### 收集同步需求

要确保类的线程安全性，就需要确保它的不变性条件不会在并发访问的情况下被破坏，这就需要对其状态进行推断。

同步策略(Synchronization Policy）定义了如何在不违背对象不变条件或后验条件的情况下对其状态的访问操作进行协同。同步策略规定了如何将不可变性、线程封闭与加锁机制等结合起来以维护线程的安全性，并且还规定了哪些变量由哪些锁来保护。

类的不变性条件与后验条件约束了在对象上有哪些状态和状态转换是有效的。在某些对象的方法中还包含一些基于状态的先验条件

在单线程程序中，如果某个操作无法满足先验条件，那么就只能失败。但在并发程序中先验条件可能会由于其他线程执行的操作而变成真。在并发程序中要一直等到先验条件为真，然后再执行该操作．

要想实现某个等待先验条件为真时才执行的操作，一种更简单的方法是通过现有库中的类实现依赖状态的行为，例如阻塞队列或者信号量。

如果以某个对象为根节点构造一张对象图，那么该对象的状态将是对象图中所有对象包含的域的一个子集。

为了防止多个线程在并发访问同一个对象时产生的相互干扰，这些对象应该要么是线程安全的对象，要么是事实不可变的对象，或者由锁来保护的对象。

#### 实例封闭

如果某对象不是线程安全的，那么可以通过多种技术使其在多线程程序中安全地使用. 你可以确保该对象只能由单个线程访问（线程封闭），或者通过一个锁来保护对该对象的所有访问。

将数据封装在对象内部，可以将数据的访问限制在对象的方法上，从而更容易确保线程在 访问数据时总能持有正确的锁。

封闭机制更易于构造线程安全的类，因为当封闭类的状态时，在分析类的线程安全时就无需检查整个程序。

从线程封闭原则及其逻辑推论可以得出 Java 监视器模式。遵循 Java 监视器模式的对象会把对象的所有可变状态都封装起来，并由对象自己的内置锁来保护。

```java
public class PrivateLock {
    private final Object lock = new Object():

    @GuardedBy("myLock") Widget widget;

    void someMethod() {
        synchronized (lock) {
            // 修改或者访问widget的状态
        }
    }
}
```

#### 线程安全性的委托

将线程安全委托给 ConcurrentMap

```java
@ThreadSafe
class DelegatingVehicleTracker {
    private final ConcurrentMap<String, Point> locations;

    private final Map<String, Point> unmodifiableMap;

    public DelegatingVehicleTracker(Map<String, Point> points) {
        locations = new ConcurrentHashMap<>(points);
        unmodifiableMap = Collections.unmodifiableMap(locations);
    }

    public Map<String, Point> getLocations() {
        return unmodifiableMap;
    }

    public Point getLocation(String id) {
        return locations.get(id);
    }

    public void setLocation(String id, int x, int y) {
        if (locations.replace(id, new Point(x, y)) == null) {
            throw new IllegalArgumentException("invalid vehicle name: " + id);
        }
    }
}
```

独立的状态变量

将线程安全性委托给名个状态变量

在鼠标事件监听器与键盘事件监听器之间不存在任何关联二者是彼此独立的，因此 VisualComponent 可以将其线程安全性委托给这两个线程安全的监听器列表

```java
class VisualComponent {

    private final List<KeyListener> keyListeners = new CopyOnWriteArrayList<KeyListener>();

    private final List<MouseListener> mouseListeners = new CopyOnWriteArrayList<>();

    public void addKeyListener(KeyListener listener) {
        keyListeners.add(listener);
    }

    public void addMouseListener(MouseListener listener) {
        mouseListeners.add(listener);
    }

    public void removeKeyListener(KeyListener keyListener) {
        keyListeners.remove(keyListener);
    }

    public void removeMouseListener(MouseListener listener) {
        mouseListeners.remove(listener);
    }
}
```

当委托失效时

如果一个类是由多个独立且线程安全的状态变量组成，养且在所有的操作中都不包含无效状态转换，那么可以将线程安全性麥托给底层的状态变量。

发布底层的状态变量

如果一个状态变量是线程安全的，并且没有任何不变性条件来约束它的值，在变量的操作上也不存在任何不允许的状态转换，那么就可以安全地发布这个变量。

#### 在现有的线程安全类中添加功能

要添加一个新的原子操作，最安全的方法是修改原始的类，但这通常无法做到，因为你可能无法访问或修改类的源代码。要想修改原始的类，就需要理解代码中的同步策略，这样增加的功能才能与原有的设计保持一致。如果直接将新方法添加到类中，那那么意味着实现同步策略的所有代码仍然处于一个源代码文件中，从而更容易理解与维护。

另一种方法是扩展这个类，假定在设计这个类时考虑了可扩展性。

扩展 Vector 井增加一个“若没有则添加” 方法

```java
class BetterVector<E> extends Vector<E> {
    public synchronized boolean putIfAbsent(E e) {
        boolean absent = !contains(e);
        if (absent)
            add(e);
        return absent;
    }
}
```

客户端加锁机制

```java

class ListHelper<E> {
    public List<E> list = Collections.synchronizedList(new ArrayList<E>());

    public boolean putIfAbsent(E e) {
        synchronized (this) {
            boolean absent = !list.contains(e);
            if (absent) {
                list.add(e);
            }
            return absent;
        }
    }
}
```

通过组合来加锁

```java
class ImprovedList<T> implements List<T> {
    private final List<T> list;

    private ImprovedList(List<T> list) {
        this.list = list;
    }

    public synchronized boolean putIfAbsent(T x){
        boolean absent = !list.contains(x);
        if (absent){
            list.add(x);
        }
        return absent;
    }

    public synchronized void clean(){list.clear();}

    // 按照类似方法实现接口中的其他方法

}
```

#### 将同步策略文档化

在文档中说明容产代码需要了解的线程安全性保证，以及代码维护员需要了解的同步策略。

如果某个类没有明确说明是线程安全的，就不要假设它是线程安全的。
