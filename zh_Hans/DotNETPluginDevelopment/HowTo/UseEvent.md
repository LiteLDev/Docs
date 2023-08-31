# 🔍 如何：使用事件

以下示例演示如何使用事件。通过订阅事件，可以快捷地获取到所需的对象与信息。

## 订阅事件

订阅一个事件的方式如下（以 [`PlayerUseItemOnEvent`](../APIs/Namespace/LLNET.Event/Class/PlayerUseItemOnEvent) 为例）：

C#
```cs
using System;
using LiteLoader.Event;

namespace PluginMain
{
    class Plugin
    {
        public static void OnPostInit()
        {
            PlayerUseItemOnEvent.Subscribe(ev => 
            {
                Console.WriteLine($"Player: {ev.Player.Name} use item on block:{ev.BlockInstance.Position}");

                //参见Tip
                ev.Dispose();

                return true;
            });
        }
    }
}
```

::: tip
及时调用`Dispose`可以小幅度地提升效率，减少CLR启用新线程析构对象带来的开销。
:::

## 以引用的方式订阅事件

以引用的方式订阅一个事件的方式如下（以 [`PlayerChatEvent`](../APIs/Namespace/LLNET.Event/Class/PlayerChatEvent) 为例）：

C#
```cs
using System;
using LiteLoader.Event;

namespace PluginMain
{
    class Plugin
    {
        public static void OnPostInit()
        {
            PlayerChatEvent.Subscribe_Ref(ev => 
            {

                ev.Message = "你猜猜我说了什么？";

                ev.Dispose();

                return true;
            });

            //下一个事件
            PlayerChatEvent.Subscribe_Ref(ev => 
            {

                //此时Message已经被修改了
                Console.WriteLine(ev.Message);

                ev.Dispose();

                return true;
            });
        }
    }
}
```

## 取消订阅事件

取消订阅十分简单，只需要保存订阅事件时返回的事件监听器实例，并在需要的时机取消订阅即可。

C#
```cs
using System;
using LiteLoader.Event;

namespace PluginMain
{
    class Plugin
    {
        public static void OnPostInit()
        {
            var listener = PlayerUseItemOnEvent.Subscribe(ev => 
            {
                ev.Dispose();
                return true;
            });

            PlayerUseItemOnEvent.Unsubscribe(listener);

            //或者直接使用listener.Remove(),效果相同。
        }
    }
}
```

## 以.NET事件的形式订阅或取消订阅事件

事件类提供了.NET风格事件，使用起来更加简单，本质上与`Subscribe`/`Unsubscribe`并无区别。

C#
```cs
using System;
using LiteLoader.Event;

namespace PluginMain
{
    class Plugin
    {
        public static void OnPostInit()
        {

            var func = (PlayerJoinEvent ev) => { Console.WriteLine(ev.Player.Name); }

            //订阅事件
            PlayerJoinEvent += func;
            //取消订阅事件
            PlayerJoinEvent -= func;

        }
    }
}
```

## 拦截事件

拦截事件同样简单，只需在回调函数中返回`false`即可。

::: tip
某些事件并不能被拦截，某些事件被拦截后可能有非预期的效果，具体事件的行为由 BDS 本身决定。
:::

C#
```cs
using System;
using LiteLoader.Event;

namespace PluginMain
{
    class Plugin
    {
        public static void OnPostInit()
        {

            PlayerEatEvent += ev =>
            {
                ev.Player.SendText("不让你吃！");
                return false;
            }

        }
    }
}
```
