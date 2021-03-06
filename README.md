# TaskFrameworkInUnity
在Unity中，原生提供的异步方法就是协程，但是协程的使用存在很多不够方便的地方。比方说，无法访问协程的返回值，而且迭代器的异常处理有一定的局限，我们不能在try-catch中加入另外一个生成器。
在支持了.Net4.x语法之后，我们可以使用一个很棒的特性，async/await异步方法。异步方法是基于Task的一个语法糖，和.Net不同，Unity对Task有自己的实现，它总是运行在主线程上面的。
单线程，支持返回值，更完整的堆栈跟踪，那么用异步方法来代替协程就成为了可能。实际上，async/await最终会被转化为若干方法和属性的动态调用，支持自定义异步行为，这也为我们基于协程实现特定的异步工作流实现了便捷。

通过实现INotifyCompletion接口，以及一些会被动态调用的属性，可以构建出一个自定义的可被等待的对象CustomAwaiter，再通过扩展方法的方式，我对一些在协程编码中常用的类型实现GetAwaiter方法，得到对应Awaiter，而这些Awaiter最终会被使用在异步工作流里面。
这些类型包括：WaitForSeconds, WaitForSecondsRealtime, WaitForEndOfFrame, WaitForFixedUpdate, AssetBundleCreateRequest, AssetBundleRequest
常见用法：
```CSharp
public async void RunTask()
{
    Debug.Log("wait 1s...");
    await new WaitForSeconds(1f);
    Debug.Log("wait fixed update");
    await new WaitForFixedUpdate();
    Debug.Log("wati end of frame");
    await new WaitForEndOfFrame();
    var ab = await AssetBundle.LoadFromFileAsync(Application.streamingAssetsPath + Path.DirectorySeparatorChar + "sphere.prefab.asset");
    Debug.Log(ab);
    var objs = await ab.LoadAssetAsync("Sphere").WrapMultiple<GameObject>();
    foreach (var obj in objs)
        Instantiate(obj);
}

```