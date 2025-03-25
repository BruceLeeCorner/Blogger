GroupBy参数最多的签名如下：

```C#
IEnumerable<TResult> GroupBy<TSource, TKey, TElement, TResult>(this IEnumerable<TSource> source, Func<TSource, TKey> keySelector, Func<TSource, TElement> elementSelector, Func<TKey, IEnumerable<TElement>, TResult> resultSelector, IEqualityComparer<TKey>? comparer);
```
---

```C#
Person[] persons = {
    new("特朗普",77),
    new("默克尔",70),
    new("马斯克",50),
    new("泽连斯基",46),
    new("李留威",30),
    new("高超",33),
    new("高梓潇",8),
    new("刘梦琪",13)
};
```

```C#
var groups = persons.GroupBy(item =>
{
    return item.Age switch
    {
        <= 14 => "1少年",
        <= 35 => "2青年",
        <= 65 => "3中年",
        _ => "4老年"
    };
});

groups = groups.OrderBy(item => item.Key);

foreach (var group in groups)
{
    foreach (var person in group)
    {
        Console.WriteLine(group.Key + "->" + person.Name);
    }
}
```

```
1少年->高梓潇
1少年->刘梦琪
2青年->李留威
2青年->高超
3中年->马斯克
3中年->泽连斯基
4老年->特朗普
4老年->默克尔
```
---

```C#
var groups = persons.GroupBy(item =>
{
    return item.Age switch
    {
        <= 14 => "1少年",
        <= 35 => "2青年",
        <= 65 => "3中年",
        _ => "4老年"
    };
}, item => new
{
    Name = item.Name,
    BirthYear = 2024 - item.Age
});

groups = groups.OrderBy(item => item.Key);

foreach (var group in groups)
{
    foreach (var elem in group)
    {
        Console.WriteLine($"{group.Key}->{elem.Name} was born in {elem.BirthYear}.");
    }
}
```

```
1少年->高梓潇 was born in 2016.
1少年->刘梦琪 was born in 2011.
2青年->李留威 was born in 1994.
2青年->高超 was born in 1991.
3中年->马斯克 was born in 1974.
3中年->泽连斯基 was born in 1978.
4老年->特朗普 was born in 1947.
4老年->默克尔 was born in 1954.
```

---

```C#
var groups = persons.GroupBy(item =>
{
    return item.Age switch
    {
        <= 14 => "1少年",
        <= 35 => "2青年",
        <= 65 => "3中年",
        _ => "4老年"
    };
}, item => new
{
    Name = item.Name,
    BirthYear = 2024 - item.Age
}, (key, items) => new { Key = key, Count = items.Count() });

groups = groups.OrderBy(item => item.Key);

foreach (var group in groups)
{
    Console.WriteLine($"{group.Key}->{group.Count}.");
}
```

```
1少年->2.
2青年->2.
3中年->2.
4老年->2.
```

---

```C#
public static void GroupByEx4()
{
    // Create a list of pets.
    List<Pet> petsList =
        new List<Pet>{ new Pet { Name="Barley", Age=8.3 },
                       new Pet { Name="Boots", Age=4.9 },
                       new Pet { Name="Whiskers", Age=1.5 },
                       new Pet { Name="Daisy", Age=4.3 } };

    // Group Pet.Age values by the Math.Floor of the age.
    // Then project an anonymous type from each group
    // that consists of the key, the count of the group's
    // elements, and the minimum and maximum age in the group.
    var query = petsList.AsQueryable().GroupBy(
        pet => Math.Floor(pet.Age),
        pet => pet.Age,
        (baseAge, ages) => new
        {
            Key = baseAge,
            Count = ages.Count(),
            Min = ages.Min(),
            Max = ages.Max()
        });

    // Iterate over each anonymous type.
    foreach (var result in query)
    {
        Console.WriteLine("\nAge group: " + result.Key);
        Console.WriteLine("Number of pets in this age group: " + result.Count);
        Console.WriteLine("Minimum age: " + result.Min);
        Console.WriteLine("Maximum age: " + result.Max);
    }

    /*  This code produces the following output:

        Age group: 8
        Number of pets in this age group: 1
        Minimum age: 8.3
        Maximum age: 8.3

        Age group: 4
        Number of pets in this age group: 2
        Minimum age: 4.3
        Maximum age: 4.9

        Age group: 1
        Number of pets in this age group: 1
        Minimum age: 1.5
        Maximum age: 1.5
    */
}
```

---

第1个委托`pet => Math.Floor(pet.Age)`，参数是Pet，返回值类型可任意，返回值相等的元素被判定成一组。仅为GroupBy()传入此实参，返回值相当于`IEnumable<EnumablePetsWithKey>`.
```
class EnumablePetsWithKey : IEnumable<Pet>
{
    public string Key {get;set;}
}
```
遍历返回值，每个元素是一个组，元素的属性Key是组的分组凭据(就是第一个委托的返回值)，元素本身也是可迭代的，迭代出的就是属于该分组的所有Pet。

上述说到：`返回值相等被认为是一组`,返回值类型的判等规则，可由GroupBy()的最后一个参数`comparer`传入。

---

第2个委托`pet => pet.Age`,输入参数是Pet，可改变组内元素的类型，默认组内元素肯定还是Pet，此委托可以将GroupBy返回的组内元素整合成其他类型。如下：
```
class EnumableSomeTypeWithKey : IEnumable<SomeType>
{
    public string Key {get;set;}
}
```
---

第3个委托

```C#
(baseAge, ages) => new
{
    Key = baseAge,
    Count = ages.Count(),
    Min = ages.Min(),
    Max = ages.Max()
});
```
参数是分组凭据和组内所有元素的集合。经过前2个委托，返回值是包含组内元素的`IEnumable<EnumablePetsWithKey>`，第3个委托相当于对EnumablePetsWithKey进行整合，比如不关心组内元素具体是什么，而是关心该组有多少元素的时候，就完全不必返回组内元素,通过聚合，返回值变成一维的了：
```
class NonEnumableObjectWithKey
{
    public string Key {get;set;}
	public object NonEnumableObject {get;set;}
}
```

当然，`完全可以同时返回组内的所有元素和该组的统计信息`：
```C#
(baseAge, ages) => new
{
    Key = baseAge,
    Ages = ages,// 组内所有元素
    Count = ages.Count(),
    Min = ages.Min(),
    Max = ages.Max()
});
```