# 编程规范

不要为构造方法的参数提供默认值，使用重载代替。

示例：

不要这么做：

`public Ctor(int a,double b = 3.14){}`

用下面的两个重载方法代替上述代码。

```csharp
public Ctor(int a)
{
    this(a,3.14);
}

public Ctor(int a,double b)
{
    _a = a;
    _b = b;
}
```







# 为类型输出格式化字符串

① 重写ToString

② 继承IConveriable

③ 继承IFormattable

④ 继承IFormatProvider,ICustomFormat

# 异常机制

## 前言

如果方法无法完成方法名所指定的任务，就应抛出一个异常。

## 异常堆栈和pdb

pdb文件内存放的是源文件路径和行号信息，如果只有dll没有pdb，ex.StackTrace只能显示调用者的命名空间和名称，无法显示行号和源文件路径，所以，我们提供别人轮子用应当dll和pdb都给到别人。

## throw和throw ex区别

```csharp
using System;
internal class Program
{
    private static void Main(string[] args)
    {
        TestThrowex();
    }

    private static void TestThrowex()
    {
        try
        {
            ThrowException();
        }
        catch (Exception ex)
        {
            throw ex;
        }
    }
    private static void ThrowException()
    {
        throw new Exception();
    }
}
```

未经处理的异常: System.Exception: 引发类型为“System.Exception”的异常。

  在 Program.TestThrowex() 位置 C:\Users\Administrator\Desktop\ConsoleApp2\ConsoleApp4\Program.cs:行号 17

  在 Program.Main(String[] args) 位置 C:\Users\Administrator\Desktop\ConsoleApp2\ConsoleApp4\Program.cs:行号 6

```csharp
using System;
internal class Program
{
    private static void Main(string[] args)
    {
        TestThrow();
    }

    private static void TestThrow()
    {
        try
        {
            ThrowException();
        }
        catch
        {
            throw;
        }
    }
    private static void ThrowException()
    {
        throw new Exception();
    }
}

```

未经处理的异常: System.Exception: 引发类型为“System.Exception”的异常。

  在 Program.ThrowException() 位置 C:\Users\Administrator\Desktop\ConsoleApp2\ConsoleApp4\Program.cs:行号 22

  在 Program.TestThrow() 位置 C:\Users\Administrator\Desktop\ConsoleApp2\ConsoleApp4\Program.cs:行号 17

  在 Program.Main(String[] args) 位置 C:\Users\Administrator\Desktop\ConsoleApp2\ConsoleApp4\Program.cs:行号 6

`结论`

1. **虽然Throw ex和Throw报告的发生异常的方法是同一个，但是Throw ex报告的并不是第一案发现场，而是catch后重新抛出的异常位置.**

2. **Throw ex会重置异常堆栈，隐藏了真正发生异常的位置。**

3. **如果方法中catch语句块重新抛出异常，请优先使用throw而不是throw ex**

## StackTrace

`方法A`抛出异常后，会在本方法内查找相应的catch,若无合适的catch，则向上抛出此异常，`方法A`的调用者`方法B`在`调用A的那一行`抛出异常，然后，方法B在本方法中查找相应的catch......就这样向上层层穿透，直到匹配到合适的catch把异常捕获或者最终抛给CLR终止程序。

同一个方法也许在多个位置抛出异常，但是在异常堆栈中这个方法只可能出现一次，报告的异常也是这个方法最后一次抛出异常的位置。

StackTrace的每一行，指明了 （1） 是哪个方法抛出异常 （2）方法在哪个文件 （3）异常发生在哪一行。

StackTrace的下一行的方法是上一行方法的调用者，下一行报告的异常行数是调用上一行方法的位置。

StackTrace的第一行是真正发生异常的方法，是最内层的方法，是异常的第一案发现场；最后一行的方法是最后一个未在方法内找到合适的异常处理程序的方法，它的调用者能catch住异常。

  **案例分析**

在 Program.ThrowException() 位置 C:\Users\Administrator\Desktop\ConsoleApp2\ConsoleApp4\Program.cs:行号 22

  在 Program.TestThrow() 位置 C:\Users\Administrator\Desktop\ConsoleApp2\ConsoleApp4\Program.cs:行号 17

  在 Program.Main(String[] args) 位置 C:\Users\Administrator\Desktop\ConsoleApp2\ConsoleApp4\Program.cs:行号 6

## try-catch-finally执行流

（1）try语句块→catch语句块

（2）try语句块→finally语句块

  (3)  try语句块→catch语句块→finally语句块  

* 一个try块必须关联至少一个catch或finally块。

* finally语句块一般用于清理非委托资源，如释放数据库连接，关闭流(文件)。

* try内是可能失败而抛出异常的语句，try能让代码不是立刻在原地终止，而是有机会去finally语句块执行清理工作，或者有机会跳转到异常处理程序。

* 如果catch抛出异常，则之前try抛出的异常会被覆盖，finally抛出异常，则之前try或catch抛出的异常会被覆盖。
* 不要再catch中清理非委托资源再把异常重新抛出，这更适合用finally实现。如果用catch，catch内外都要有清理代码，而且还要捕获再抛出异常，太过繁琐。

## 利用IL解释异常为什么损害程序性能

<img src="D:\OneDrive\mdimg\image-20220425115510279.png" alt="image-20220425115510279" style="zoom:67%;" />

![image-20220425115519303](D:\OneDrive\mdimg\image-20220425115519303.png)

带有try-catch-finally结构的方法编译后的IL代码都会附带一个异常处理表，表的每一行标记了哪个地址范围的指令抛出什么类型的异常后去哪个地址范围执行异常处理程序，catch越多，表行数越多。所以，当方法在运行时，抛出异常后需要去查表，从而损害了程序性能，但如果未抛出异常则没必要查表，对程序的损害就微乎其微。

## try-catch-finally的return规则

请判断打印结果Console.WriteLine(GetResult().Age);

 ```csharp
 private static Person GetResult()
 {
   int a = 1, b = 2;
   Person p = new() { Age = 1 };
   try
   {
      int k = a / b;
      return p;
   }
   catch (Exception ex)
   {
      Console.WriteLine(ex.Message);
      throw;
   }
   finally
   {
      p.Age++;
   }
 }
 private class Person
 {
   public int Age { get; set; }
 }
 ```

答案：2

 请判断打印的结果 Console.WriteLine(GetResult());

```csharp
private static int GetResult()
{
    int a = 1, b = 2, n = 1;
    try
    {
        int k = a / b;
        return n;
    }
    catch(Exception ex)
    {
        Console.WriteLine(ex.Message);
        throw;
    }
    finally
    {
        n++;
    }
}
```

答案：1

**解析**

try catch finally结构，try catch 运行到return时，先计算return后表达式的结果，然后将结果(必定是一个存储在栈中的引用地址或值类型的本身值)存储到暂存区，再去执行finally语句块，最后回到try或catch中将暂存区的结果返回。

前者，暂存的是一个指向堆中的实例的引用地址，暂存时，p.Age等于1，执行finally后，p.Age等于2，所以最终返回p时，p.Age等于2。强调，执行finally前后，暂存区的引用地址并无变化，无论是否执行finally，返回的值相同。

后者，暂存时，会现在栈中开辟一个变量存储值类型本身而非地址，相当于暂存了一份n的副本，finally让n++，并不会影响到暂存的副本，所以最终返回1.

这道测试题，考察的是引用类型和值类型的内存模型的区别。

 # 线程同步

## 取放料协同

取料完成后，立刻放料；放料完成后立刻取料，初始化时要先放料。

不推荐轮询标志位决定何时取料或放料，这样会浪费CPU。

使用AutoRestEvent，取料和放料相互通知。

```csharp
private static void Main(string[] args)
{
    AutoResetEvent putEvent = new AutoResetEvent(true); // 放料锁初始状态要放行，设置成true
    AutoResetEvent fetchEvent = new AutoResetEvent(false);

    Task.Run(() => PutCoordinately(putEvent, fetchEvent));
    Task.Run(() => FetchCoordinately(putEvent, fetchEvent));
}

private static void PutCoordinately(AutoResetEvent putEvent, AutoResetEvent fetchEvent)
{
    while (true)
    {
        if (putEvent.WaitOne())
        {
            Put();
            fetchEvent.Set();
        }
    }
}

private static void FetchCoordinately(AutoResetEvent putEvent, AutoResetEvent fetchEvent)
{
    while (true)
    {
        if (fetchEvent.WaitOne())
        {
            Fetch();
            putEvent.Set();
        }
    }
}

private static void Put()
{
    Thread.Sleep(1000);
    Console.WriteLine("put materials...");
}

private static void Fetch()
{
    Console.WriteLine("fetch marterials...");
}
```

## 打印10次ABC

一个线程打印A，一个线程打印B，一个线程打印C，期望在控制台打印10次ABC，即ABCABCABCABCABCABCABCABCABCABC.

```csharp
private static void Main(string[] args)
{
    AutoResetEvent eventA = new AutoResetEvent(true);
    AutoResetEvent eventB = new AutoResetEvent(false);
    AutoResetEvent eventC = new AutoResetEvent(false);

    Task.Run(() => Print(eventA, eventB, 'A'));
    Task.Run(() => Print(eventB, eventC, 'B'));
    Task.Run(() => Print(eventC, eventA, 'C'));
}

private static void Print(AutoResetEvent a, AutoResetEvent b, char @char)
{
    for (int i = 0; i < 10; i++)
    {
        if (a.WaitOne())
        {
            Console.WriteLine(@char);
            b.Set();
        }
    }
}
```

三个线程协同，使用三个AutoResetEvent，AutoResetEvent既可以阻塞线程，又可以通知线程放行。

# aysnc 和 await

异步方法：耗时长交期长，可以先触发其开始做，不需要结果，只要一份对结果的"承诺合同"，再需要的时候，凭借“承诺合同”，索取结果。

同步方法：



设计一个方法时，如果方法















异步方法。

同步方法。











流程控制图分为单引擎和无限引擎











流程控制： 顺序结构，选择结构，循环结构，异步结构。

只有前3项的结构是一维的流程，仅考虑是否执行，只需要一个驱动引擎；包含4项的结构是二维的流程，不仅考虑到是否执行，还考虑到执行的耗时。











假设都是同步方法会怎样？？？

工作者线程，非工作者线程。



await做了什么？

封 - 接 - 返

所以，执行流一定能像promise和sync那样运行。万事开个头。

一个流程，其实是方法A调用方法B,方法B再调用方法C，，，



异步方法和同步方法。



1. 开发准则



异步方法的优点：

人的角度：

解决一个大问题，首先要将一个大问题分解成多个小问题，按照小问题之间的依赖关系决定的顺序，逐个去解决每个小问题，当所有的小问题都被解决了，大问题就自然被解决了。

每个小问题对应一个解决它的小方法，将小方法按照小问题之间的依赖关系决定的顺序拼装起来，就得到一个解决问题的执行流程。如果有个引擎驱动这个执行流程，问题就被解决了。

假设CPU是无限多的。

这些小方法耗时不一，如果耗时长的方法，能够支持先

书写流程控制时，就是组装各种各样的方法。如果交期长的

多个CPU执行方法：

同步方法，耗时短交期短，结果立等可取，现场稍等片刻与离开再回去要相比更划算。

异步方法，

编写流程(组合方法)时，按照人的思维，先触发新的异步方法，再去执行同步方法，需要的时候阻塞

计算机角度

既能按照字面上的流程执行，又能保证无线程阻塞，又能同时让多个线程忙起来。

如果遵照同步开发和异步开发的准则去写方法，那么能保证，没有任何一个线程，因为等待异步方法的结果而阻塞。





租用线程。



异步方法：被调用的时候开始去做，但没做完便返回，返回的不是结果，而是结果的一份“承诺”，就像"合同"。调用者











程序员书写代码的目的就是得到一个这样的执行流程。程序员要关注的是执行流的顺序对不对，执行流执行的快不快，毕竟我们都期望问题在最短的时间内按照正确的顺序执行完毕得出正确的结果。

方法调用的顺序对不对，如果每个方法都是同步方法，自然是对的，但是存在一个问题。有的方法执行的慢，有的方法执行的快，



每个要分步骤，每个步骤就是一个小方法，最终将很多小方法拼装起来

每个小问题对应一个解决它的小方法。







方法只是一种执行流，租用线程执行，但不要霸占线程，自己不用的时候，放弃租用，让线程给别人用。但方法异步次数要尽量少，因为太多了，线程不够用，只能搞更多的线程，线程数远远高于CPU核心数，导致上下文切换频繁，增大了额外的开销，导致得不偿失。





调度器安排任务到线程上执行，并不是上下文切换，上下文切换是指线程从CPU上撤下来。

 







为什么要有异步方法？

租用线程，而非霸占线程。方法的异步性是种执行流，按照执行流走，至于哪个线程驱动它，不重要。只要我们保证线程没有

实现异步编程，让应用程序能够具有并发性。最终达到的目的满足以下两种要求：

（1）尽可能多的线程处于繁忙状态和待调度状态，尽可能少的线程因为等待另一个方法的结果而处于阻塞状态。
（2）保持线程数量不多于CPU的核心数，避免切换上下文开销太大，同时也不少于CPU核心数，充分利用CPU。





假设有5个同步方法，每个同步方法耗时1s，主线程上的Main方法调用这5个同步方法

```csharp
public static void Main(string[] args)
{
    SyncMethod1();
    SyncMethod2();
    SyncMethod3();
    SyncMethod4();
    SyncMethod5();
}
```

<img src="D:\OneDrive\viaennote\imgs\image-20220714220931217.png" alt="image-20220714220931217" style="zoom:50%;" />

性能分析：占用1个线程，0次线程上下文切换，总耗时5秒。





调用10个同步方法，只有同步方法执行完毕，调用者拿到结果，才能往下走。如果同步方法耗时很久，

为什么要有异步方法？

## 异步编程套路

### 异步方法，也可以称允诺方法

一个Service，就是一个API，就是一个Method。

一个耗时短交期短的Service，应该被设计成一个同步方法。用户提出服务诉求后，用户呆在原地稍等片刻，同步方法现场交付服务结果。

一个耗时久交期长的Service，应该被设计成一个异步方法，它向用户允诺：用户提出服务诉求后，服务并非立等可取。用户可以先拿着一纸合同离开去忙自己的其他事宜。用户在未来的某个时刻，发现无服务无法展开自己的剩余工作，就可以凭借合同索取服务结果。

一个使用了异步方法的方法也应该被设计成异步方法，就像一个服务如果驱动了一个允诺服务，自身也必然是允诺服务。

允诺方法有什么好处呢？

（1）用户可以做其他事情，转轮询等待为需要时去要。

（2）用户索要服务时，如果未完成，就等待，但这种等待不占用任何线程资源。服务完成了，自然激活用户。



一个异步方法，内部必须调用了异步方法，或新建了Task，否则，都不算是异步方法，更不应该加async.



如何编写一个异步方法？

1. 确认方法类型是异步方法还是同步方法
   * 如果方法存在耗时长交期长的操作，则是承诺方法。
   *  如果方法需要调用异步方法，则是承诺方法。
2. 确认方法签名
   * 方法若无输出，则返回类型是`Task`
   * 方法若有输出，则返回类型是`Task<输出类型>`
   * 方法签名：Task<输出类型> 方法名(参数列表)
3. 将代码块分成多个能够并发执行的小块，为耗时长的小块启动一个Task执行
4. 调用异步方法(如果需要的话)，但暂时不`await` ，只有后续必须获取异步方法的执行结果时才去`await`
5. 保证在方法末尾，方法中的每一个Task引用(方法创建的Task的引用 +  异步方法返回的Task引用)，都被直接返回或被`await`

经典的异步方法案例如下：

当方法中存在两个及以上的Task引用时，推荐直接用await

案例一

```csharp

async Task<输出类型> 方法名 (输入参数列表)
{
      Task t1 = Task.Run(()=>{ 耗时长的小块1 })
      Task t2 = Task.Run(()=>{ 耗时长的小块2 })
      Task t... = Task.Run(()=>{ 耗时长的小块... })
      Task tn = Task.Run(()=>{ 耗时长的小块n })
          
      耗时短的小块1;
      耗时短的小块2;
      耗时短的小块...;
      耗时短的小块n;
          
      await Task.WhenAll(t1,t2,...,tn);
 }
```



案例二

```csharp
Task<输出类型> 方法名 (输入参数列表)
{
    Task t = Task.Run(()=>{ 耗时长的小块1 });
    return t;
    // await t; 这样也可以，但是建议优先选择直接返回Task引用，效率会更高
}
```



案例三

```csharp
Task<输出类型> 方法名 (输入参数列表)
{
    Task t = MethodAsync();
    // DO Somesthing
    return t;
    // await t; 这样也可以，但是建议优先选择直接返回Task引用，效率会更高
}
```



案例四

```csharp
Task<输出类型> 方法名 (输入参数列表)
{
    Task t = MethodAsync();
    Task t2 = Task.Run(()=>{ 耗时长的小块1 });
    // DO Somesthing
    
    return t;
    // await t; 这样也可以，但是建议优先选择直接返回Task引用，效率会更高
}
```



**规则**

（1）保证在异步方法结束前，方法中的每一个Task引用(方法新建的Task的引用 +  异步方法返回的Task引用)，都被直接返回或被`await`

（2）既不关心返回值也不关心失败成功的Task引用，不需要被返回或await

（3）不需要获取结果的Task引用，不要去await，那这种Task引用只能作为返回值返回，如果类型不匹配，那说明Task不合理，您要去改。



规则的目的：异步方法是一个承诺，承诺是否被履行可以通过异步方法的返回值的Task引用获取。如果异步方法存在既没有await也没有被返回的Task引用，那么异步方法的承诺存在漏洞，结局就是异步方法承诺的事情的履行结果并不能全部的被用户知道。

**规范**

1. 越早调用异步方法越好，越迟await异步方法越好。
2. 越早启动封装耗时长的代码块的Task越好，越迟await越好。
3. 如果方法不需要获取异步方法和新建的任务的方法，那就不要await。
4. 没有await的Task引用必须作为返回值返回，除非Task引用与方法签名的返回值类型不一致，或者由于方法只能接受一个Task返回值，存在多余的不能作为返回值的Task引用的情况下，那就await这些Task引用，即使你此刻并不需要这些Task的最终执行结果。
5. 耗时长的操作开Task,耗时短的不要开Task。
6. 永远不要试图中断异步方法链（中断异步方法的传染特征），永远不要试图直接或间接的调用Task.Wait()，永远不要使用Thread.Sleep()，虽然您这样做程序也能正常运行，但这样做会降低程序的并发性能。

规范的目的：最大可能性的让异步方法能够在程序中充分利用线程，发挥出最大的并发能力。

**疑问**

如果我await的越晚，有可能我做了很多的事情，因为异步方法失败而白费？

如果出现这种情况，程序的重点问题并不是并发性了，而是转移成为什么你的方法总是失败。失败带来性能开销，往往远远大于白费。







****

**耗时久，这个久到底指多长时间呢？**

（3）

将操作分为可并行的两部分，耗时长，耗时短。

Task t = Task.Run(耗时长);

耗时短。

await t;



如果调用了异步方法，建议先驱动异步方法，执行一段其他事情，再去等待。



如果异步方法中，新开了很多Task,要保证任何一个Task引用都被await或作为返回值！







而且，每一个Task都是一个允诺，异步方法必须在自己的尾部，对所有的允诺负责，把允诺合同返回，不能漏下任何一个允诺。



如何评判一个服务耗时多少算久？

大于启动1个新线程和1次切换CPU线程的时间算久。个人感觉5us,大于0.1ms的可以开个新的Task.















































多线程的最终目的：使用最少的线程数，在一定时间内，完成更多的任务。





餐馆 - 线程池

服务员 - 线程

顾客点餐 - 任务

<img src="D:\OneDrive\mdimg\image-20220712180829736.png" alt="image-20220712180829736" style="zoom: 33%;" />

web服务器接收http请求，转发请求给asp net core，紧接着去处理其他请求。一段时间后，web程序将结果计算完毕，web服务器自动去将响应回馈给客户端。





await 和waitone() 不同，前者放弃当前线程，waitone（）不放弃当前线程。



异步方法：返回值是Task或Task<T>的方法。

① 方法名以Async结尾

② 返回值是Task或Task<T>，不建议使用void

③ 异步方法具有传染性。





不要太在意async，其实本来无需async的，只是await关键字是后期发明的，为了兼容以前的程序，不把await作为变量，所以加async标志await作为关键字。



**推荐使用异步API,不要使用同步API**

NET 推荐HttpClient，HttpWebClient已经被淘汰。





学会调用异步lambda表达式。



当你的lambda方法需要调用异步方法时，为lambda加async





你的任务能够被同步的流畅处理，只是帮你做事的人可能不同。







aync能改变方法的返回值的书写形式。事实上，C#会new一个task，且直接把task的状态置为已完完成且不接受CPU调度，然后把result赋值成返回值，作为方法的返回值。





async的好处是避免了回调函数写法的地域模式和线程间状态交换的困难。



asynchronous的缺点：异步方法生成状态机的类，肯定比直接返回task慢。







在异步方法中不要使用Sleep  用task.delay











**解释：没有await也可以async,但是有await必须要有async**



**演示：异步方法被同步调用Wait(),那异步方法还起到多线程的优化好处吗**



**从微软推荐尽量调用异步API，建议多多使用异步方法，保证异步方法链不要断**



**有一个Task<string>的异步方法，在await后，开启了两个Task，那么怎么方法内部如何写返回值呢？**



**await 一个线程时，到底发生了什么？**



**await用的太多了会怎样？**





var t = Task.run();

await t;



和 



var t = Task.run();

return t；



有什么分别？
