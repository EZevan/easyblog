# C# - System.Uri.GetComponents()
> Gets the specified components of the current instance using specified escaping for special characters.   
> 使用特殊字符的指定转义，获取当前实例的指定部分

```C#
public string GetComponents (UriComponents components, UriFormat format);
```

**参数**  
> **components** UriComponents   
[UriComponents](https://docs.microsoft.com/zh-cn/dotnet/api/system.uricomponents?view=netcore-3.1)值的按位组合，它指定当前实例中要返回给调用方的部分

**参数**  
> **format** UriFormat   
[UriFormat](https://docs.microsoft.com/zh-cn/dotnet/api/system.uriformat?view=netcore-3.1)枚举值之一，用来控制如何转义字符

**用法示例: 展示UriComponents各个部分的值**
```C#
var uri = new Uri("https://evan.zhang@www.sologolo.com:8088/subject/index?id=123456");

var components = enum.GetValues(typeof(UriComponents));
foreach(var component in components)
{
    Console.WriteLine($"{value} = {uri.GetComponents(value,UriFormat.UriEscaped)}\n");
}
/*输出结果为如下：
Scheme = https
UserInfo = evan.zhang
Host = www.sologolo.com
Port = 8088
SchemeAndServer = https://evan.zhang@www.sologolo.com:8088
Path = subject/index
Query = id=123456
PathAndQuery = /subject/index?id=123456
HttpRequestUrl = https://evan.zhang@www.sologolo.com:8088/subject/index?id=123456
Fragment =
AbsoluteUri = https://evan.zhang@www.sologolo.com:8088/subject/index?id=123456
StrongPort = 8088
HostAndPort = www.sologolo.com:8088
StrongAuthority = evan.zhang@www.sologolo.com:8088
NormalizedHost = www.sologolo.com
KeepDelimiter =
SerializationInfoString = https://evan.zhang@www.sologolo.com:8088/subject/index?id=123456
*/
```

**用法示例：使用“|”按位或运算符拼接组件**
```C#
var uri = new Uri("https://evan.zhang@www.sologolo.com/subject/index?id=123456");

var uriString = uri.GetComponents(UriComponents.SchemeAndServer | UriComponents.UserInfo,UriFormat.Unescaped)

Console.WriteLine(uriString);
// 输出结果为：https://evan.zhang@www.sologolo.com
```