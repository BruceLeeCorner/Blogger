每个SQL语句执行结束会返回一张或多张二维表，每个SQL可以使用下面3个方法的任何1个来执行，但是在实际使用时，会根据需求选择其中最合适的1个执行SQL。

不关心返回的二维表可选择1；只想获取第一张表(即使返回多张表)的第一个Cell数据，可选择2；如果想获取第一张表的所有CELL数据或所有表的所有CELL的数据，可选择3.

1. ExecuteNonQuery()
2. ExecuteScalar()
3. ExecuteReader()

**dynamic转换表的行记录**

读取dynamic的属性时，属性名称必须与返回表的字段名称完全一致，包括大小写，否则会异常。

```csharp
NpgsqlConnection connection = new NpgsqlConnection("Host=localhost;Username=postgres;Password=123456;Database=postgres");
string sql = """
SELECT 'Jack' "namE", 18 "AgE"
""";
var employees = connection.Query(sql).ToList();
Console.WriteLine(employees[0].namE);
Console.WriteLine(employees[0].AgE);
```

**Model Class属性名称与返回表字段名称大小写关系**

Query*方法，返回表的每一个字段映射Model一个属性，返回表的每一行映射一个Model实例。如果表字段名称与Model属性名称相同(`忽略大小写`)，则映射成功，该字段的值会赋值给此属性；如果Model Class的属性和表字段都不匹配(允许Model Class存在不匹配的属性，甚至全部的属性都不匹配都不会异常，Query*返回的列表的Model实例个数仍旧和表的行记录数量相等)，此属性的值不会被表字段赋值，而是Model Class构造函数执行完毕后的值，一般是属性数据类型的默认值。

Note：返回表的列名不要重复，可在SQL语句使用AS显示指定列名来达到不重复的目的。

```csharp
public class StandardDog
{
    public Guid Id { get; set; }
    public string Name { get; set; }
    public int? Age { get; set; }
    public int IgnoredProperty
    { get { return 1; } }
}
```

```csharp
NpgsqlConnection connection = new NpgsqlConnection("Host=localhost;Username=postgres;Password=123456;Database=postgres");
string sql = """
    SELECT "id", "name", "age" FROM "dog";
    """;
var dogs = connection.Query<StandardDog>(sql).ToList();
```

dogs的每个实例的`IgnoredProperty`属性的值都是1.

**执行带参数的SQL**

在SQL语句中，字面值可来自变量，变量用@xxx形式表示，注意，不管@xxx表示的是何种类型,@xxx不需要用引号包裹。有两种传入实参的方式，一种是利用匿名类型，一种是利用DynamicParameters.
匿名类型，成员名称是去掉@后的xxx,大小写要一致。Dapper根据成员的C#类型推断出字段的DbType，然后用合适的形式替换SQL语句，如SELECT @id;如果是new {id = 1},则SELECT 1;如果是new{id="1"},则SELECT '1';
SQL语句往往根据不同的筛选条件动态拼接，@参数不固定，那么匿名类型的成员的数量也要动态变化，但是匿名类型是做不到的。这时候需要用DynamicParameters.

*匿名类型*

```csharp
using(IDbConnection connection = new NpgsqlConnection("Host=localhost;Username=postgres;Password=123456;Database=postgres"))
{
    string sql = """
    SELECT @name AS "Name", @age AS "Age"
    """;

    var employees = connection.Query(sql, new { name = "Jack", age = 18 }).ToList();
    Console.WriteLine(employees[0].Name);
    Console.WriteLine(employees[0].Age);
}
```

*DynamicParameters*

```csharp
// SELECT @param1, @param2;
var queryParams = new DynamicParameters();
queryParams.Add("param1", dynamicValue1, System.Data.DbType.Guid);
queryParams.Add("param2", dynamicValue2, System.Data.DbType.String);
```