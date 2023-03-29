# <center>C# - PropertyInfo.GetValue方法

> 命名空间: System.Reflection   
> 程序集: System.Runtime.dll   
> 官方定义: 返回指定对象的属性值   
> 详细说明可参考官方文档:   
https://docs.microsoft.com/zh-cn/dotnet/api/system.reflection.propertyinfo.getvalue?view=netcore-3.1

**注：由于静态属性（成员）属于类型本身（随着类的加载而存在），而不属于单个对象；因此通过将null作为object参数传递来获取静态属性值**   
* 以下示例表示获取NUnit.Framework.TestContext类的静态属性CurrentContext  

```C#
PropertyInfo propertyInfo = typeof(TestContext).GetProperty("CurrentContext");
Console.WriteLine("Currentcontext: " + propertyInfo.GetValue(null));
```