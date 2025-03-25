# Container的内部结构

Container内部维护一个映射表，如下

Type1 + key1 --> List<Func<SimpleContainer, object>>

Type1 + key2 --> List<Func<SimpleContainer, object>>

Type1 + key3 --> List<Func<SimpleContainer, object>>
......

Type1 + keyN --> List<Func<SimpleContainer, object>>

Type2 + key1 --> List<Func<SimpleContainer, object>>

Type3 + NULL --> List<Func<SimpleContainer, object>>

Type3 + Key1 --> List<Func<SimpleContainer, object>>

Type和Key作为键，构造Type实例的的委托列表是值，每一次Register会增加一个映射对或在已存在的映射对的值中追加一个构造Type实例的委托。

# 如何理解注册

1. Type和Key不存在，会增加一个映射对。
2. Type和Key已存在，把一个构造Type实例的方法Append到List中。
3. 重复注册相同的Type+Key(Type和Key都相同)是允许的，只是会导致该Type和Key组合对应的构造Type实例委托会有多个。重复注册几次就有几个构造委托，且允许构造委托完全相同的逻辑或写法。
4. RegisterSingleton,RegisterPerRequest,RegisterInstance这3种注册方式其实都是通过调用RegisterHandler实现的。仅使用RegisterHandler就可以实现所有的注册方式！
5. void RegisterHandler(Type service, string key, Func<SimpleContainer, object> handler) => this.GetOrCreateEntry(service, key).Add(handler);

# HasHandler(Type service, string key)
功能：判断 [Type + Key] 是否已经被注册过。

Key非NULL: Type + Key，有则true，无则false.  Key是NULL：曾经注册过Type + 任意Key则true,否则false.

# ContainerEntry GetEntry(Type service, string key)

功能：返回映射对

Key非NULL: 返回Type + Key，无则返回NULL.   Key是NULL：注册过Type + NULL，则返回Type + NULL，否则返回Type + Key1（最先被注册的Type + 非NULL Key）或 NULL

# object GetInstance(Type service, string key)

相同的Type和Key被注册多次，调用GetInstance会导致抛出异常，因为存在多个委托，但是只索要一个实例，那么就存在歧义，SimpleContainer也不知道您到底想要哪一个。这是SimpleContainer的特点，而其他依赖注入框架如Microsoft.Extensions.DependencyInjection会返回最后一次追加的委托返回的实例。



---------------------------------------------------------------------------------

# 如何选择择构造函数

1. 构造函数必须是public
2. 优先选择参数的类型被注册到容器的数量最多的构造函数。
3. 未注册的类型的参数将使用NULL值。
4. 参数的实例是GetInstance(type,null)获取的。
5. Ienumable<T>通过调用GetAllInstances<Type,NULL>获取的。




ContainerEntry: 
List<delete>的派生类，是集合。内部的元素是输入参数是容器，输出参数是服务实例的委托。key和type和count一样，是集合的固有属性，会为每个元素打上烙印。相当于Dictionary<key & type,List<delete>>的一个元素。

集合作为成员的类可以考虑使用继承，KeyValuePair<Key,Value> = class ContainerEntry : List<T> {key}.







ContainerEntry GetEntry(Type service, string key)：

(1) Type未注册返回NULL.(未调用过任何Register(Type,任意值Key))
          // Type注册了
(2) Key不是NULL时，注册了返回，未注册返回NULL。
(3) Key为NULL时，若注册了Type + NULL则返回，若未注册Type + NULL，在诸多Type + KeyN中选择最先注册的Type + Key1. 

 非NULL: Type + Key;     NULL：Type + NULL  Or  Type + Key1 of Type + KeyN

 



GetInstance
key不是 null: 找Type和Key完全相等的ContainerEntry. 找不到匹配的返回null
key是null: 如果注册了key = null，返回null对应的ContainerEntry。 未注册key=null, 返回非null key.



未指定key的注册，相当于注册了 key = null的ContainerEntry。
重复注册相同key和type，注册方法不会抛出任何异常。会在列表中添加一个委托。但是这会导致GetInstance 抛出异常！！！GetInstance
寻找到key和Type对应的ContainerEntry，然后如果委托元素有多个，则会直接抛出异常。
GetAllInstances运行正常。会遍历运行所有的委托，有几个委托就返回几个实例。

RegisterSingleton和RegisterPerRequest没什么区别。

如何避免同一个类型和key多次被注册？？？
用Handler。

多个构造函数时，会选择哪个构造函数？
构造函数如何选择函数参数的构造函数，不传入，直接用null，用的是第一次key.


同一个类型，注册不同的key的场景？

如何用List实现Dictionary<key,List<T>> ?





1. 单例
2. 多个单例
3. 每次请求都会生成一个新的实例
4. 接口只注册一种服务
5. 接口注册多种具体服务