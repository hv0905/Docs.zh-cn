---
title: 使用 ASP.NET Core SignalR 中流式处理
author: tdykstra
description: ''
monikerRange: '>= aspnetcore-2.1'
ms.author: tdykstra
ms.custom: mvc
ms.date: 06/07/2018
uid: signalr/streaming
ms.openlocfilehash: 0001eed830249ac46ba35331759187bb4e7e8fd3
ms.sourcegitcommit: 3ca527f27c88cfc9d04688db5499e372fbc2c775
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/17/2018
ms.locfileid: "39095257"
---
# <a name="use-streaming-in-aspnet-core-signalr"></a>使用 ASP.NET Core SignalR 中流式处理

通过[brennan，第 Conroy](https://github.com/BrennanConroy)

ASP.NET Core SignalR 支持流式处理服务器方法的返回值。 这是适用于其中的数据片段会随时间的方案。 当返回值流式传输到客户端时，每个片段是发送到客户端变得可用，而非等待所有的数据变得可用。

[查看或下载示例代码](https://github.com/aspnet/Docs/tree/live/aspnetcore/signalr/streaming/sample)（[如何下载](xref:tutorials/index#how-to-download-a-sample)）

## <a name="set-up-the-hub"></a>设置中心

集线器方法自动成为流式处理的集线器方法时它将返回`ChannelReader<T>`或`Task<ChannelReader<T>>`。 下面是一个示例，显示的数据流式传输到客户端的基础知识。 每当将对象写入到`ChannelReader`该对象将立即发送给客户端。 在结束时，`ChannelReader`完成告诉客户端流已关闭。

> [!NOTE]
> 写入`ChannelReader`在后台线程并返回`ChannelReader`越早越好。 将阻止其他集线器调用，直到`ChannelReader`返回。

[!code-csharp[Streaming hub method](streaming/sample/Hubs/StreamHub.cs?range=10-34)]

## <a name="net-client"></a>.NET 客户端

`StreamAsChannelAsync`方法`HubConnection`用于调用流式处理方法。 将中心方法名称和到中心方法中定义的自变量传递`StreamAsChannelAsync`。 上的泛型参数`StreamAsChannelAsync<T>`指定流式处理方法返回的对象的类型。 一个`ChannelReader<T>`返回从流调用，并且表示在客户端上的流。 若要读取数据，常见模式是循环访问`WaitToReadAsync`，并调用`TryRead`数据时可用。 循环将结束时该流已关闭服务器，或取消标记传递给`StreamAsChannelAsync`被取消。

```csharp
var channel = await hubConnection.StreamAsChannelAsync<int>("Counter", 10, 500, CancellationToken.None);

// Wait asynchronously for data to become available
while (await channel.WaitToReadAsync())
{
    // Read all currently available data synchronously, before waiting for more data
    while (channel.TryRead(out var count))
    {
        Console.WriteLine($"{count}");
    }
}

Console.WriteLine("Streaming completed");
```

## <a name="javascript-client"></a>JavaScript 客户端

JavaScript 客户端调用集线器上流式处理方法使用`connection.stream`。 `stream`方法接受两个参数：

* 集线器方法的名称。 在以下示例中，中心方法名称是`Counter`。
* 在集线器方法中定义的参数。 在以下示例中，参数是： 若要接收的流项和流项之间的延迟的数量的计数。

`connection.stream` 返回`IStreamResult`其中包含`subscribe`方法。 传递`IStreamSubscriber`到`subscribe`并设置`next`， `error`，并`complete`回调以获取来自通知`stream`调用。

[!code-javascript[Streaming javascript](streaming/sample/wwwroot/js/stream.js?range=19-36)]

若要结束的客户端调用的流`dispose`方法`ISubscription`从返回`subscribe`方法。

## <a name="related-resources"></a>相关资源

* [中心](xref:signalr/hubs)
* [.NET 客户端](xref:signalr/dotnet-client)
* [JavaScript 客户端](xref:signalr/javascript-client)
* [发布到 Azure](xref:signalr/publish-to-azure-web-app)
