# 🔩 SecondaryRankPtr 结构

## 定义

<br>

> 用于表示二级指针的结构。

<br>

C#
```cs
public struct SecondaryRankPtr
```
C++
```cpp
public value class SecondaryRankPtr
``` 

<br>


继承 [Object](https://docs.microsoft.com/zh-cn/DotNET/api/system.object) → [ValueType](https://docs.microsoft.com/zh-cn/DotNET/api/system.valuetype) → [SecondaryRankPtr](SecondaryRankPtr)

<br>

## 构造函数
- 
    |||
    |-|-|
    |`SecondaryRankPtr(void**)`|用二级 [void](https://docs.microsoft.com/zh-cn/DotNET/api/system.void) 指针 初始化 `SecondaryRankPtr` 结构的新实例。|
    |`SecondaryRankPtr(IntPtr)`|用 [System.IntPtr](https://docs.microsoft.com/zh-cn/DotNET/api/system.intptr) 作为二级指针 初始化 `SecondaryRankPtr` 结构的新实例。|

<br>

## 方法
- 
    |||
    |-|-|
    |[ToSecPointer()]|返回当前二级指针。|
    |[ToPointer()]|返回解引用后的一级指针。|
    |[ToIntPtr()]|返回解引用后的一级指针的IntPtr形式。|


