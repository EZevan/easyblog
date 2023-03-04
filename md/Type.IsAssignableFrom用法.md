# C# Type.IsAssignableFrom(Type type)

> 官方解释：Determine whether an instance of current Type can be assigned from an instance of the specified type.   
判断指定类型实例是否可以分配给当前类型的实例

* 可以用来判断指定类型是否继承了当前类  
```C#
public class Car
{
    public void Run()
    {
        Console.Writeline("Running...");
    }
}

public class Audi : Car
{
    
}

public class Bmw
{

}

static void Main(string[] args)
{
    var carType = typeof(Car);
    var audiType = typeof(Audi);
    var bmwType = typeof(Bmw);

    Console.Writeline($"Audi is drived from Car? {carType.IsAssignableFrom(audiType)}");  // true;
    Console.Writeline($"Bmw is drived from Car? {carTyep.IsAssignableFrom(bmwType)}")   // false;
}
```

* 可以用来判断匿名类型是否继承了当前类
```C#
static void Main(string[] args)
{

    Console.Writeline($"Audi is drived from Car? {Check(typeof(Audi))}");   // true;
    Console.Writeline($"Bmw is drived from Car? {Check(typeof(Bmw))}");    // false;

    private bool Check<T>(T t)
    {
        return typeof(Car).IsAssignableFrom(t.GetType());
    }
}
```
* 可以用来判断匿名类型是否为当前类
```c#
static void Main(string[] args)
{
    var carType = typeof(Car);
    var audiType = typeof(Audi);
    var bmwType = typeof(Bmw);

    Console.Writeline(carType.IsAssignableFrom(carType));   // true;
    Console.Writeline(audiType.IsAssignableFrom(audiType)); // true;
    Console.Writeline(bmwTupe.IsAssignableFrom(bmwTupe));   // true;
}
```
