---
title: Asynchronous Saving | Microsoft Docs
author: rowanmiller
ms.author: divega
ms.date: 01/24/2017
ms.assetid: b64a606e-ecd9-4807-829a-b6ec05ade33f
ms.technology: entity-framework-core
uid: core/saving/async
---

# Asynchronous Saving

异步保存避免在更改写入数据库时阻塞线程。这可能是有用的，以避免富客户端应用程序的UI卡死。异步操作还可以增加web应用程序中的吞吐量，当数据库操作完成时，可以释放线程以服务其他请求。更多信息，见[Asynchronous Programming in C#](https://msdn.microsoft.com/en-us/library/mt674882.aspx)。

Asynchronous saving avoids blocking a thread while the changes are written to the database. This can be useful to avoid freezing the UI of a thick-client application. Asynchronous operations can also increase throughput in a web application, where the thread can be freed up to service other requests while the database operation completes. For more information, see [Asynchronous Programming in C#](https://msdn.microsoft.com/en-us/library/mt674882.aspx).

>[!警告]
> EF核心不支持在同一个上下文实例上运行的多个并行操作。在开始下一个操作之前，您应该始终等待操作完成。这通常是通过在每个异步操作上使用`await`关键字来完成的。

> [!WARNING]
> EF Core does not support multiple parallel operations being run on the same context instance. You should always wait for an operation to complete before beginning the next operation. This is typically done by using the `await` keyword on each asynchronous operation.

Entity Framework Core 提供`DbContext.SaveChangesAsync()`作为`DbContext.SaveChanges()`的替代。

Entity Framework Core provides `DbContext.SaveChangesAsync()` as an asynchronous alternative to `DbContext.SaveChanges()`.

````csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;

namespace EFSaving.Async
{
    public class Sample
    {
        public static async Task RunAsync()
        {
            using (var db = new BloggingContext())
            {
                await db.Database.EnsureDeletedAsync();
                await db.Database.EnsureCreatedAsync();
            }

            await AddBlogAsync("http://sample.com");
        }

        #region Sample
        public static async Task AddBlogAsync(string url)
        {
            using (var db = new BloggingContext())
            {
                var blog = new Blog { Url = url };
                db.Blogs.Add(blog);
                await db.SaveChangesAsync();
            }
        }
        #endregion
    }
}
````