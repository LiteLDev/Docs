# 🔍 如何：使用静态命令

以下示例演示如何使用静态命令。静态命令相较于动态命令具有更好的可读性，其编写方式也相对比较简单。不像动态命令，静态命令采用了声明式的注册方法。

## 声明命令类主体

声明命令类主体的方式如下：

C#
```cs
// command.cs

using System;
using LiteLoader.DynamicCommand;

namespace Example;

// 须继承ICommand接口

[Command("examplecmd")]
public class ExampleCommand: ICommand
{
    public void Execute(CommandOrigin origin, CommandOutput output)
    {
        
    }
}
```

其中`CommandAttribute`为命令类提供了命令名称这一信息，继承`ICommand`接口并实现`Execute`方法为命令类提供了回调函数，也就是命令的运行逻辑主体。

让我们来添加更多东西！

C#
```cs
// command.cs

using System;
using LiteLoader.DynamicCommand;

namespace Example;

[Command("examplecmd", Description = ".NET example command", Permission = CommandPermissionLevel.Any)]
public class ExampleCommand: ICommand
{
    public void Execute(CommandOrigin origin, CommandOutput output)
    {
        
    }
}
```

此时的`CommandAttribute`提供了命令名称、命令描述、命令权限。将命令权限设置为`CommandPermissionLevel.Any`，将使得所有玩家都可以使用此命令。

## 声明命令枚举

在声明命令主体的基础上，声明命令枚举的方式如下：

C#
```cs
// command.cs

using System;
using LiteLoader.DynamicCommand;

namespace Example;

[Command("examplecmd", Description = ".NET example command", Permission = CommandPermissionLevel.Any)]
public class ExampleCommand: ICommand
{

    // 使用CommandEnumAttribute声明命令枚举
    [CommandEnum]
    enum ExampleEnum{ add, remove, list };

    public void Execute(CommandOrigin origin, CommandOutput output)
    {
    }
}
```

## 定义命令参数

有了之前所作的准备工作，此时我们可以开始定义命令参数。相关示例如下：

C#
```cs
// command.cs

using System;
using LiteLoader.DynamicCommand;

namespace Example;

[Command("examplecmd", Description = ".NET example command", Permission = CommandPermissionLevel.Any)]
public class ExampleCommand: ICommand
{

    [CommandEnum]
    enum ExampleEnum{ add, remove, list };

    // 使用CommandParameterAttribute定义命令参数
    [CommandParameter(ParamType.Enum)]
    ExampleEnum Mode{ get; set; }

    [CommandParameter(ParamType.Int)]
    int Count{ get; set; }

    public void Execute(CommandOrigin origin, CommandOutput output)
    {
        Console.WriteLine($"Mode: {Mode},Count: {Count}");
    }
}
```

## 注册命令

>使用DynamicCommand.RegisterCommand\<TCommand\>方法注册命令。

C#
```cs
// command.cs

using System;
using LiteLoader.DynamicCommand;

namespace Example;

[Command("examplecmd", Description = ".NET example command", Permission = CommandPermissionLevel.Any)]
public class ExampleCommand: ICommand
{

    [CommandEnum]
    enum ExampleEnum{ add, remove, list };

    //使用CommandParameterAttribute定义命令参数
    [CommandParameter(ParamType.Enum)]
    ExampleEnum Mode{ get; set; }

    [CommandParameter(ParamType.Int)]
    int Count{ get; set; }

    public void Execute(CommandOrigin origin, CommandOutput output)
    {
        Console.WriteLine($"Mode: {Mode},Count: {Count}");
    }
}
```
```cs
// plugin.cs

using LiteLoader.DynamicCommand;
using Example;

namespace PluginMain
{
    class Plugin
    {
        public static void OnPostInit()
        {
            DynamicCommand.RegisterCommand<ExampleCommand>();
        } 
    }
}
```

# 进阶

>以下为进阶内容。使用这些内容可以编写出功能更为强大的命令。

## 设置命令重载

>**不同的命令参数组合可形成命令的不同重载形式**。每一个命令参数都具有一个或多个命令重载标识。若未在 `CommandParameterAttribute` 中指明 `OverloadId` 属性，则命令参数的默认重载标识为 `0` 。使用 `CommandOverloadIdAttribute` 可以为参数指明多个重载标识。标识可以为任意 `Int32` 值，LL.NET将会把具有相同重载表示的命令参数添加到同一个命令重载中。

>以下示例基于前面的内容演示如何使用命令重载：

C#
```cs
// command.cs

using System;
using LiteLoader.DynamicCommand;

namespace Example;

[Command("examplecmd", Description = ".NET example command", Permission = CommandPermissionLevel.Any)]
public class ExampleCommand: ICommand
{

    [CommandEnum]
    enum ExampleEnum{ add, remove, list };

    [CommandParameter(ParamType.Enum, 
     OverloadId = 0,
     /*使用CommandParameterOption.EnumAutocompleteExpansion将枚举参数展开*/
     Option = CommandParameterOption.EnumAutocompleteExpansion
     )]
    [CommandParameterOverload(1)]

    ExampleEnum Mode{ get; set; }

    [CommandParameter(ParamType.Int, OverloadId = 1)]

    int Count{ get; set; }

    //   此时的命令重载列表
    //   /examplecmd <add|remove|list>
    //   /examplecmd <add|remove|list> <int>
    //   当然，也可以将list单独作为一个枚举声明，在此只是作为演示。

    public void Execute(CommandOrigin origin, CommandOutput output)
    {
        switch(this.Mode)
        {
            case ExampleEnum.add:
            {
                Console.WriteLine("added. Count:" + Count.ToString());
            }
            break;
            case ExampleEnum.remove:
            {
                Console.WriteLine("removed. Count:" + Count.ToString());
            }
            break;
            case ExampleEnum.list:
            {
                Console.WriteLine("listed.")
            }
            break;
        }
    }
}
```
```cs
// plugin.cs

using LiteLoader.DynamicCommand;
using Example;

namespace PluginMain
{
    class Plugin
    {
        public static void OnPostInit()
        {
            DynamicCommand.RegisterCommand<ExampleCommand>();
        } 
    }
}
```

## 设置空命令重载

>有时，命令并不需要设置任何参数，如BDS中的 /list /stop 等命令。对此，可以选择对其设置空命令重载以满足需求。

>以下示例演示如何使用空命令重载：

C#
```cs
using System;
using LiteLoader.DynamicCommand;
using MC;

namespace Example;

[Command("killallplayers")]

//使用CommandEmptyOverloadAttribute指明此命令具有空重载
[CommandEmptyOverload]

public class KillAllPlayersCommand: Icommand
{
    public void Execute(CommandOrigin origin, CommandOutput output)
    {
        Level.RunCmdEx("kill @a");
    }
}

//省略注册过程...
```

>同时， `CommandEmptyOverloadAttribute` 与一般的命令重载兼容，在此不再赘述。

## 设置可选参数

>可让命令参数作为可选参数供使用者决定是否输入。

>以下示例演示如何设置可选参数：

C#
```cs
using System;
using LiteLoader.DynamicCommand;

namespace Example;

[Command("examplecmd")]

//使用CommandEmptyOverloadAttribute指明此命令具有空重载
[CommandEmptyOverload]

public class ExampleCommand: Icommand
{

    //设置为可选
    //若参数未在本次调用中被设置，则其值为上一次设置的值或默认值
    //现阶段并未实现查询参数是否被设置的方法
    //使用命令重载时可能会导致同样的状况
    [CommandParameter(ParamType.Int, IsMandatory = false)]
    int Test{ get; set; }

    public void Execute(CommandOrigin origin, CommandOutput output)
    {
    }
}
```

## 获取命令注册信息

>通过继承ICommandEvent或ICommandData，可获取到命令注册前后的动态命令实例与内部表示该命令的数据。

### 继承ICommandEvent

C#
```cs
using System;
using LiteLoader.DynamicCommand;

namespace Example;

[Command("examplecmd")]
public class ExampleCommand: Icommand, ICommandEvent
{
    public void Execute(CommandOrigin origin, CommandOutput output)
    {
    }

    //实现接口方法
    //在方法结束后动态命令实例便会被销毁
    //若想长期保存该实例，请使用DynamicCommandInstance.Release通用方法释放原始指针，并使用构造函数 DynamicCommandInstance(global::DynamicCommandInstance* p) 重新构造该实例。

    public void BeforeCommandSetup(DynamicCommandInstance cmd)
    {
        //do something...
    }

    //setup之后的实例只建议进行查询等操作，试图调用改变当前实例的方法可能会造成一些未知的效果或错误。（未经测试）

    public void AfterCommandSetup(DynamicCommandInstance cmd)
    {
        //do something...
    }
}
```

### 继承ICommandData

>继承此接口可获取到内部表示的命令注册信息。获取更多信息请查阅 [LiteLoader.DynamicCommand.Internal] 命名空间。

C#
```cs
using System;
using LiteLoader.DynamicCommand;

namespace Example;

[Command("examplecmd")]
public class ExampleCommand: Icommand, ICommandEvent
{
    public void Execute(CommandOrigin origin, CommandOutput output)
    {
    }

    //实现接口方法
    //在方法结束后动态命令实例便会被销毁
    //若想长期保存该实例，请使用DynamicCommandInstance.Release通用方法释放原始指针，并使用构造函数 DynamicCommandInstance(global::DynamicCommandInstance* p) 重新构造该实例。

    public void BeforeCommandSetup(CommandManager.CommandData cmdData, DynamicCommandInstance cmd)
    {
        //do something...
    }

    //setup之后的实例只建议进行查询等操作，试图调用改变当前实例的方法可能会造成一些未知的效果或错误。（未经测试）

    public void AfterCommandSetup(CommandManager.CommandData cmdData, DynamicCommandInstance cmd)
    {
        //do something...
    }
}
```
