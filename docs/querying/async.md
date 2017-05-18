---
title: Asynchronous Queries | Microsoft Docs
author: rowanmiller
ms.author: divega
ms.date: 01/24/2017
ms.assetid: b6429b14-cba0-4af4-878f-b829777c89cb
ms.technology: entity-framework-core
uid: core/querying/async
---

# Asynchronous Queries

异步查询避免在数据库中执行查询时阻塞线程。这可能是有用的，以避免富客户端应用程序的UI假死。异步操作还可以增加web应用程序中的吞吐量，当数据库操作完成时，可以释放线程以服务其他请求。有关更多信息，参见[Asynchronous Programming in C#](https://msdn.microsoft.com/en-us/library/mt674882.aspx)。

Asynchronous queries avoid blocking a thread while the query is executed in the database. This can be useful to avoid freezing the UI of a thick-client application. Asynchronous operations can also increase throughput in a web application, where the thread can be freed up to service other requests while the database operation completes. For more information, see [Asynchronous Programming in C#](https://msdn.microsoft.com/en-us/library/mt674882.aspx).

> [!WARNING]

> EF Core不支持在同一个上下文实例上运行的多个并行操作。在开始下一个操作之前，您应该始终等待操作完成。通常的做法是在每个异步操作前加`await`关键字。

> EF Core does not support multiple parallel operations being run on the same context instance. You should always wait for an operation to complete before beginning the next operation. This is typically done by using the `await` keyword on each asynchronous operation.

Entity Framework Core提供了一套异步扩展方法，可以作为一种替代使用，造成查询被执行并返回结果的LINQ方法。例如包括`ToListAsync()`、`ToArrayAsync()`、 `SingleAsync()`等。没有异步版本的LINQ操作符如 `Where(...)`、 `OrderBy(...)`等。因为这些方法只建立LINQ表达式树并不会导致查询在数据库中执行。

Entity Framework Core provides a set of asynchronous extension methods that can be used as an alternative to the LINQ methods that cause a query to be executed and results returned. Examples include `ToListAsync()`, `ToArrayAsync()`, `SingleAsync()`, etc. There are not async versions of LINQ operators such as `Where(...)`, `OrderBy(...)`, etc. because these methods only build up the LINQ expression tree and do not cause the query to be executed in the database.

> [!IMPORTANT]

> EF Core异步扩展方法是在`Microsoft.EntityFrameworkCore`命名空间中定义。必须为可用的方法导入此命名空间。

> The EF Core async extension methods are defined in the `Microsoft.EntityFrameworkCore` namespace. This namespace must be imported for the methods to be available.

<!--[!code-csharp[Main](../../../samples/core/Querying/Querying/Async/Sample.cs#Sample)]-->
```csharp
public async Task<List<Blog>> GetBlogsAsync()
{
    using (var context = new BloggingContext())
    {
        return await context.Blogs.ToListAsync();
    }
}
```
