

接收到包的类

过滤器，过滤出包

自定义Appserver

# IP字符串转整数



# 连接过滤器

摘要：禁止部分ip连接服务器，或者说只允许指定的ip才可以连接服务器。

1. 观察接口IConnectionFilter
    Name：可以为连接过滤器起个名称，Name的赋值发生在Initialize()里面完成
    AllowConnect()：返回true，表示此ip允许连接；false，表示此ip禁止连接
    Initialize()：可以直接返回true，此函数的有两个作用（1）Name赋值 （2）从配置文件读取允许连接的ip以供使用
```C#
    //
    // 摘要:
    //     The basic interface of connection filter
    public interface IConnectionFilter
    {
        //
        // 摘要:
        //     Gets the name of the filter.
        string Name { get; }

        //
        // 摘要:
        //     Whether allows the connect according the remote endpoint
        //
        // 参数:
        //   remoteAddress:
        //     The remote address.
        bool AllowConnect(IPEndPoint remoteAddress);
        //
        // 摘要:
        //     Initializes the connection filter
        //
        // 参数:
        //   name:
        //     The name.
        //
        //   appServer:
        //     The app server.
        bool Initialize(string name, IAppServer appServer);
    }
```
2. 实现一个继承IConnectionFilter的连接过滤器
```C#
public class IPConnectionFilter : IConnectionFilter
{
    /// <summary>
    /// 可以用一个线性表或者HashSet的私有字段存放允许连接/禁止连接的ip集合，该集合在Initialize()填充
    /// </summary>
    List<string> ipsAllowed;
    public string Name { get; private set; }

    //返回true，表示允许连接。返回false，表示禁止连接。
    public bool AllowConnect(IPEndPoint remoteAddress)
    {
        string ip = remoteAddress.Address.ToString();
        return ipsAllowed.Contains(ip);
    }
    /// <summary>
    /// 1. 实现为该连接过滤器赋值
    /// 2. 从配置文件或者代码实现，将允许或不允许连接的ip加载到ipsAllowed中，以供AllowConnect()使用
    /// </summary>
    /// <param name="name">连接过滤器名称</param>
    /// <param name="appServer">可以适用任何服务器</param>
    /// <returns></returns>
    public bool Initialize(string name, IAppServer appServer)
    {
        Name = name;
        ipsAllowed = new List<string>();
        ipsAllowed.Add("192.168.0.1");
        ipsAllowed.Add("172.168.13.111");
        return true;
    }
}
```
3. 向AppServer的ConnectionFilters添加过滤器
```C#
AppServer server = new AppServer();
server.ConnectionFilters.Append(new IPConnectionFilter());
```

