# 相对路径

相对路径是针对当前工作目录(Present Working Directory)而言的，即，相对当前工作目录的路径。

当前工作目录：.\

当前工作目录的上级目录： ..\

当前工作目录的上级的上级目录：..\..\

打开当前工作目录下的文件test.txt

File.Open("./test.txt");

Console.Write("./test.txt");（这是错误的）

```C#
System.IO.Directory.GetCurrentDirectory()
System.IO.Directory.SetCurrentDirectory()
```

获取或设置当前工作目录。

```
this.GetType().Assembly.Location
```

获取程序运行时，查找到的dll的所在目录





# 获取应用程序根目录的最佳写法

```c#
string _path = System.Diagnostics.Process.GetCurrentProcess().MainModule.FileName; // bin/debug/app.exe
_path = Path.GetDirectoryName(_path); // bin/debug/
```

