# 引入params关键字的目的

如果方法的参数是一维数组，可以利用params关键字，简化用户调用时传递实参的写法。

 如下API

 ```csharp
 void Test(int a, double[] doubles);
 ```
 调用方式

 ```csharp
 Test(0,new double[]{1.1,2.2,3.3});
 ```

 缺点：调用时，必须键入`new type[ ]`，这样在参数数组的元素数量只有两三个时，显得很麻烦。

 现在利用params关键字优化一下API来避免上述缺点。

 ```csharp
 void Test(int a, params double[] doubles);
 ```

 新的调用方式可以是且只能是以下2种方式之一。

 ```csharp
 Test(0,1.1,2.2,3.3); // 方式 1
 
 Test(0,new double[]{1.1,2.2,3.3}); // 方式 2
 ```

 ==**采用方式1调用API，会很舒服！这就是引入params关键字的作用。**==

# 重难点：params object[ ] args

不同情况下，编译生成的代码可能不同，难点就是如何判断出正确的编译后的等价代码形式。强调一点：==params编译后会被去除。==

编译完成后，不再存在`void Test(params type[] args)`，它被编译成`void Test(type[] args)`.



**情况1：type不是object时**

调用形式必须是以下2种情况:

① obj1, obj2, obj3会被封装到作为实参的数组。

```csharp
type obj1;
type obj2;
type obj3;
Test(obj1,obj2,obj3); // 等价 Test(new type[] { obj1,obj2,obj3});
```

②

```csharp
type[] objs = new type[n];
Test(objs);
```

编译时，检查到实参是和形参是类型匹配的数组，就不进行任何处理，直接作为实参。



**情况2：type是object时**

```c#
void Invoke(params object[] list)；
```

下面两种调用方式产生的结果完全不同！！！  

```csharp
int[] myIntArray = { 5, 6, 7, 8, 9 };
object[] myobjectArray = { 5, 6, 7, 8, 9 };

Invoke(myIntArray);    // list仅有1个元素，new object[] {myIntArray}
Invoke(myobjectArray); // list有5个元素,new object[]{5,6,7,8,9}
```

==请仔细分析下面这种情况，否则极易出错==

```c#
void Hi<T>(T arg)
{
    Invoke(arg);
}

Hi(myobjectArray);
```

上述代码片段的`list`实际仅有一个元素，是`new object[]{myobjectArray}`.

原因是编译后的代码是下面这样的：

```c#
// 编译后的代码
void Hi<T>(T arg)
{
    Invoke(new object[] {arg});
}
// 运行时生成
Hi<object[]>(object[] arg)
{
    Invoke(new object[] {arg});
}
```

# 规则

- **params修饰在参数的前面且参数类型得是一维数组类型**

  回归到C#加入params关键字的初衷：`当方法的参数是一维数组时，加params修饰简化传实参的写法。`

- **不允许ref或out params数组**

  ```csharp
  public void Test(ref params int [] ints) // 编译失败
  public void Test(out params int [] ints) // 编译失败
  ```

- **params 数组必须是方法的最后一个参数**

  ```csharp
  void Test(params double[] doubles,int a); // 编译失败
  ```

- **只能有一个params数组参数.**

  ```csharp
  void Test(int a, params double[] doubles, params int[] ints); // 编译失败
  ```

- **params修饰的参数数组可以不传入实参，但实参不是NULL而是长度为0的空数组。**

  ```csharp
  void Test(int a, params double[] doubles);
  
  Test(1); // doubles != null 且 doubles.Length 等于0
  ```

- **不能给params参数数组提供默认值，但允许传入NULL作为实参，但要注意在方法内部处理NULL的情况。**

  ```csharp
  void Test(int a, params double[] doubles = null); // 编译失败
  ```

  ```csharp
  void Test(int a, params double[] doubles)
  {
      if(doubles == null)
      {
           // DO Something
      }
      else
      {
          // DO Something
      }
  }
  
  Test(1,null); // 编译成功，运行成功，允许的操作
  ```

- **通过`ILSpy`反编译的源码我们可以看到params是一个语法糖，其实只是为了增加编程效率，本质在编译的时候会被具体的声明的数组类型替代，不参与到运行时，也就是编译后的IL代码并不存在params。**

  ```csharp
  void Test(int a, params double[] doubles);
  编译后生成的方法签名是
  void Test(int a, double[] doubles);
  ```

- **调用后会自动生成数组包装参数。**

  ```csharp
  Test(0,1.1,2.2,3.3);
  本质是
  Test(0,new double[]{1.1,2.2,3.3});
  ```

- **params关键字不构成方法的签名的一部分,不能重载一个只基于params关键字的方法。**

  ```csharp
  void Test(int a, params double[] doubles);
  void Test(int a, double[] doubles); 
  // 编译失败，这两个方法不能构成重载。因为我们知道前者编译后会去掉params，得到的签名与后者完全相同。
  ```

- **非params方法总是优先于一个params方法。**

  ```csharp
  void Test(int a, params double[] doubles);
  void Test(int a, double double1,double double2);
  
  Test(0,1.1,2.2); // 运行时调用的是后者。原则就是找最清晰最贴切的API。
  ```

- **如果在项目中绝大部分场景传递给参数数组的实参数量很少，根据二八原则，可以定义含有0，1，2，3个参数的方法进行重载，避免数组分配，提高效率。**

  ```csharp
  void Test(int a, params double[] doubles);
  void Test(int a);
  void Test(int a, double double1);
  void Test(int a, double double1,double double2);
  void Test(int a, double double1,double double2,double double3);
  
  Test(0);Test(0,1);Test(0,1,2);Test(0,1,2,3);都没调用params方法，没有构造数组。
  ```

- **派生类继承params方法时可以删除params，仍旧是正确的继承。原因就是编译后params被去除，删除params后派生类和基类的方法签名仍旧是一致的。**

  ```csharp
  interface I
  {
      void Test(params int[] args);
  }
  
  class II : I
  {
      void Test(int[] args);
  }
  ```

- **可以随时删除方法的params或为方法添加params，不影响程序中其他调用该方法的地方，调用处编译不报错表示语义没变，若编译报错去改调用处就行了。**

  ```csharp
  如C# BCL中作为所有委托类型的基类的Delegate，它的DynamicInvoke方法签名如下：
  Delegate.DynamicInvoke(params object[] @objects);
  其实如果微软把它设计成
  Delegate.DynamicInvoke(object[] @objects);
  也是没有任何问题的。
  ```

- **MSDN官方讲解示例**

  ```csharp
  public class MyClass
  {
      public static void UseParams(params int[] list)
      {
          for (int i = 0; i < list.Length; i++)
          {
              Console.Write(list[i] + " ");
          }
          Console.WriteLine();
      }
  
      public static void UseParams2(params object[] list)
      {
          for (int i = 0; i < list.Length; i++)
          {
              Console.Write(list[i] + " ");
          }
          Console.WriteLine();
      }
  
      static void Main()
      {
          // You can send a comma-separated list of arguments of the
          // specified type.
          UseParams(1, 2, 3, 4);
          UseParams2(1, 'a', "test");
  
          // A params parameter accepts zero or more arguments.
          // The following calling statement displays only a blank line.
          UseParams2();
  
          // An array argument can be passed, as long as the array
          // type matches the parameter type of the method being called.
          int[] myIntArray = { 5, 6, 7, 8, 9 };
          UseParams(myIntArray);
  
          object[] myObjArray = { 2, 'b', "test", "again" };
          UseParams2(myObjArray);
  
          // The following call causes a compiler error because the object
          // array cannot be converted into an integer array.
          //UseParams(myObjArray);
  
          // The following call does not cause an error, but the entire
          // integer array becomes the first element of the params array.
          UseParams2(myIntArray);
      }
  }
  /*
  Output:
      1 2 3 4
      1 a test
  
      5 6 7 8 9
      2 b test again
      System.Int32[]
  */
  ```