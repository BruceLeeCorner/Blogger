```C#
interface I { }
interface II : I { }
class III : II { }
class IIII : III { }
```

**IsSubclassOf 判断一个类型是不是继承自指定的类型**

```C#
// 不适用于接口
typeof(IIII).IsSubclassOf(typeof(I)); // False 
// 不适用于接口
typeof(II).IsSubclassOf(typeof(I)); // False  
// 不适用于接口
typeof(IIII).IsSubclassOf(typeof(II)); // False  

typeof(IIII).IsSubclassOf(typeof(III)); // True 
// 相同类型返回False
typeof(IIII).IsSubclassOf(typeof(IIII)); // False
```

**IsAssignableFrom()**

```C#
typeof(I).IsAssignableFrom(typeof(I)); // True

typeof(I).IsAssignableFrom(typeof(II)); // True

typeof(I).IsAssignableFrom(typeof(III)); // True
 
typeof(III).IsAssignableFrom(typeof(IIII)); // True

typeof(IIII).IsAssignableFrom(typeof(IIII)); // True
```





// 判断某个接口或类是否继承了某个接口
Array.IndexOf(typeof(IIII).GetInterfaces(), typeof(I))