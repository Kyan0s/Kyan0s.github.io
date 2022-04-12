## Unity 逆向学习记录 - 某 native plugin 对回调的处理

<br>

### 0. 废话

分析某游戏时发现其有了一堆供 native plugin 调用的回调函数，但却找不到具体是哪些 native 函数调用了这些回调。分析了下发现该程序在初始化时将回调函数地址一股脑喂给 native 函数，需要调用回调的 native 函数在调用时直接查找地址进行调用。虽然自知少见多怪但觉得还是蛮有意思的，索性记录之。

<br>

### 1. C# 部分

某游戏有多个提供给 native plugin 的回调函数（以下仅以 log 相关函数为例）：

```cs
[MonoPInvokeCallback(typeof(ClassName.LogHandler))]
private static void LogMessage(string message) {}

[MonoPInvokeCallback(typeof(ClassName.LogHandler))]
private static void LogWarning(string warning) {}

[MonoPInvokeCallback(typeof(ClassName.LogHandler))]
private static void LogError(string error) {}
```

这些回调函数被包裹在如下结构体中：

```cs
private struct InitializationParameters {
    [MarshalAs(UnmanagedType.FunctionPtr)]
    public ClassName.LogHandler LogMessage;
    
    [MarshalAs(UnmanagedType.FunctionPtr)]
    public ClassName.LogHandler LogWarning;
    
    [MarshalAs(UnmanagedType.FunctionPtr)]
    public ClassName.LogHandler LogError;
}
```

初始化时喂给 native 函数：

```cs
[DllImport("my_dll", CharSet = CharSet.Ansi, EntryPoint = "my_initialize")]
private static extern MyEvent Initialize(ClassName.InitializationParameters parameters);

public static bool Initialize() {
    ClassName.InitializationParameters parameters = new ClassName.InitializationParameters {
        LogMessage = new ClassName.LogHandler(ClassName.LogMessage),
        LogWarning = new ClassName.LogHandler(ClassName.LogWarning),
        LogError = new ClassName.LogHandler(ClassName.LogError)
    };
    MyEvent myEvent = ClassName.Initialize(parameters);
    bool flag = myEvent.code == 0;
    return flag;
}
```
<br>

### 2. Native 部分

简而言之，`my_initialize` 函数画风如下：

```c
*(undefined8 *)0x180fd9220 = *(undefined8 *)(arg2 + 0x18);
*(undefined8 *)0x180fd9228 = *(undefined8 *)(arg2 + 0x20);
*(undefined8 *)0x180fd9240 = *(undefined8 *)(arg2 + 0x28);
*(undefined8 *)0x180fd9238 = *(undefined8 *)(arg2 + 0x30);
*(undefined8 *)0x180fd9230 = *(undefined8 *)(arg2 + 0x38);
```

调用回调：

```c
void fcn.1807f2be0(uintmax_t arg1) {
    if (*(code **)0x180fd9200 != (code *)0x0) {
        if (0xf < *(uint64_t *)(arg1 + 0x18)) {
            arg1 = *(uintmax_t *)arg1;
        }
        (**(code **)0x180fd9200)(arg1);
        return;
    }
    return;
}
```

<br>
