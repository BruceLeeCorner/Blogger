##  IsGenericType



## IsGenericTypeDefinition

## GetGenericArguments()









## 如何判断一个类型是可空值类型Nullable<>

```c#
TypeInfo genericTypeInfo = typeof(T).GetTypeInfo();

if (genericTypeInfo.IsValueType)
{
    if (
        (!genericTypeInfo.IsGenericType) 
        || (!typeof(Nullable<>).GetTypeInfo().IsAssignableFrom(genericTypeInfo.GetGenericTypeDefinition().GetTypeInfo()))
       )
    {
        throw new InvalidCastException(Resources.DelegateCommandInvalidGenericPayloadType);
    }
}

```









DisplayTypeInfo(typeof(G<>));
 static void DisplayTypeInfo(Type t)
{
    Console.WriteLine("\r\n{0}", t);
    Console.WriteLine("\tIs this a generic type definition? {0}",
        t.IsGenericTypeDefinition);
    Console.WriteLine("\tIs it a generic type? {0}",
        t.IsGenericType);
    Type[] typeArguments = t.GetGenericArguments();
    Console.WriteLine("\tList type arguments ({0}):", typeArguments.Length);
    foreach (Type tParam in typeArguments)
    {
        Console.WriteLine("\t\t{0}", tParam);
    }
}



## 如何理解泛型和继承关系

```c#
class God<T> { }

God<int>; God<string>; God
```















# 范型的好处

1.算法复用,减少代码量.

2.避免频繁拆箱和装箱,提高运行效率.

3.在集合中,不用频繁向上和向下转型,降低犯错概率.

4.在集合中,最能体现范型的价值.

# 使用范型的技巧

程序中大量使用范型,或范型参数的个数较多时,代码看起来会比较凌乱.比如

```
/*通讯录 <姓名，手机号列表>*/
Dictionary<string, List<string>> contact = new Dictionary<string, List<string>>();
```

我们可以这样优化:

```
class Contact : Dictionary<string, List<string>> { }
Contact contact = new Contact();
```

Contact类无需任何实现,它的能力继承自Dictionary<string, List<string>>,二者完全等价. 这对理解继承思想有很大帮助.