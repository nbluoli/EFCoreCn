---
title: Client vs. Server Evaluation | Microsoft Docs
author: rowanmiller
ms.author: divega

ms.date: 10/27/2016

ms.assetid: 8b6697cc-7067-4dc2-8007-85d80503d123
ms.technology: entity-framework-core
 
uid: core/querying/client-eval
---
# 客户端 vs. 服务端面评估
# Client vs. Server Evaluation

Entity Framework Core支持评估查询那部分在客户端，那部分推送到数据库。数据库提供者将评估决定那部分查询在数据库。

Entity Framework Core supports parts of the query being evaluated on the client and parts of it being pushed to the database. It is up to the database provider to determine which parts of the query will be evaluated in the database.

> [!TIP]
>你可以在GitHub上查看文本的[示例项目](https://github.com/aspnet/EntityFramework.Docs/tree/master/samples/core/Querying)。
> You can view this article's [sample](https://github.com/aspnet/EntityFramework.Docs/tree/master/samples/core/Querying) on GitHub.

## 客户端面评估
## Client eval

在下面的示例中，有一个辅助方法应用于标准化从SQL Server 数据库返回blogs的Urls。因为SQL Server 数据库没办法理解这个方法是怎么实现，它也不可能翻译成SQL。所有其他方面的查询评估是在数据库中，但通过返回`URL`处理这个方法是执行在客户端上。

In the following example a helper method is used to standardize URLs for blogs that are returned from a SQL Server database. Because the SQL Server provider has no insight into how this method is implemented, it is not possible to translate it into SQL. All other aspects of the query are evaluated in the database, but passing the returned `URL` through this method is performed on the client.

<!-- [!code-csharp[Main](samples/core/Querying/Querying/ClientEval/Sample.cs?highlight=6)] -->
````csharp
var blogs = context.Blogs
    .OrderByDescending(blog => blog.Rating)
    .Select(blog => new
    {
        Id = blog.BlogId,
        Url = StandardizeUrl(blog.Url)
    })
    .ToList();
````

<!-- [!code-csharp[Main](samples/core/Querying/Querying/ClientEval/Sample.cs)] -->
````csharp
public static string StandardizeUrl(string url)
{
    url = url.ToLower();

    if (!url.StartsWith("http://"))
    {
        url = string.Concat("http://", url);
    }

    return url;
}
````
## 禁用客户评估
## Disabling client evaluation

虽然客户端评估是非常有用的，但在某些情况下，它可能会导致性能差。请考虑下面的查询，其中辅助方法现在被使用在筛选器中。因为不能在数据库中执行，所有数据都被拉入内存，然后过滤器就被应用在客户端上。导致性能不佳取决于数据量，以及数据的多少被过滤掉。

While client evaluation can be very useful, in some instances it can result in poor performance. Consider the following query, where the helper method is now used in a filter. Because this can't be performed in the database, all the data is pulled into memory and then the filter is applied on the client. Depending on the amount of data, and how much of that data is filtered out, this could result in poor performance.

<!-- [!code-csharp[Main](samples/core/Querying/Querying/ClientEval/Sample.cs)] -->
````csharp
var blogs = context.Blogs
    .Where(blog => StandardizeUrl(blog.Url).Contains("dotnet"))
    .ToList();
````

默认情况下，商户端执行评估时，EF Core将记录一个警告。查看[Logging](../miscellaneous/logging.html)查看日志输出的更多信息。您可以更改客户端评估时发生的行为，要么抛出或不做任何事情。你可以在自己的上下文选项设置——通常在`DbContext.OnConfiguring`，如果您使用的是ASP.NET Core,则可以`Startup.cs`中修改配置。

By default, EF Core will log a warning when client evaluation is performed. See [Logging](../miscellaneous/logging.html) for more information on viewing logging output. You can change the behavior when client evaluation occurs to either throw or do nothing. This is done when setting up the options for your context - typically in `DbContext.OnConfiguring`, or in `Startup.cs` if you are using ASP.NET Core.

<!-- [!code-csharp[Main](samples/core/Querying/Querying/ClientEval/ThrowOnClientEval/BloggingContext.cs?highlight=5)] -->
````csharp
protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
{
    optionsBuilder
        .UseSqlServer(@"Server=(localdb)\mssqllocaldb;Database=EFQuerying;Trusted_Connection=True;")
        .ConfigureWarnings(warnings => warnings.Throw(RelationalEventId.QueryClientEvaluationWarning));
}
````