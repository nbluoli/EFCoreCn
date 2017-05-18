---
title: Tracking vs. No-Tracking | Microsoft Docs
author: rowanmiller
ms.author: divega

ms.date: 10/27/2016

ms.assetid: e17e060c-929f-4180-8883-40c438fbcc01
ms.technology: entity-framework-core

uid: core/querying/tracking
---
# Tracking vs. No-Tracking

跟踪行为控制Entity Framework Core是否会在其更改跟踪器中保留实体实例的信息。如果一个实体被跟踪了，任何变化被发现，实体将在`SaveChanges()`时，都保存到数据库中。Entity Framework Core也会将导航属性应用到跟踪查询实体和先前通过DbContext实例从数据库加载出来的实体上。

Tracking behavior controls whether or not Entity Framework Core will keep information about an entity instance in its change tracker. If an entity is tracked, any changes detected in the entity will be persisted to the database during `SaveChanges()`. Entity Framework Core will also fix-up navigation properties between entities that are obtained from a tracking query and entities that were previously loaded into the DbContext instance.

> [!TIP]
> 你可以在GitHub查看文本的[示例项目](https://github.com/aspnet/EntityFramework.Docs/tree/master/samples/core/Querying)。

> You can view this article's [sample](https://github.com/aspnet/EntityFramework.Docs/tree/master/samples/core/Querying) on GitHub.

## Tracking queries

默认情况下，返回实体类型的查询将被跟踪。这意味着你可以改变这些实体实例和改变持续`SaveChanges()`被跟踪。

By default, queries that return entity types are tracking. This means you can make changes to those entity instances and have those changes persisted by `SaveChanges()`.

在下面的例子中，blogs rating的变更会被发现以及在`SaveChanges()`时被持续保存到数据库。

In the following example, the change to the blogs rating will be detected and persisted to the database during `SaveChanges()`.

<!-- [!code-csharp[Main](samples/core/Querying/Querying/Tracking/Sample.cs)] -->
````csharp
using (var context = new BloggingContext())
{
    var blog = context.Blogs.SingleOrDefault(b => b.BlogId == 1);
    blog.Rating = 5;
    context.SaveChanges();
}
````

## No-tracking queries

当结果只使用在只读场景下不跟踪查询是非常有用。它们将非常快速的执行，因为它不需要设置变更跟踪信息。

No tracking queries are useful when the results are used in a read-only scenario. They are quicker to execute because there is no need to setup change tracking information.

可以将单个查询替换为无跟踪：

You can swap an individual query to be no-tracking:

<!-- [!code-csharp[Main](samples/core/Querying/Querying/Tracking/Sample.cs?highlight=4)] -->
````csharp
using (var context = new BloggingContext())
{
    var blogs = context.Blogs
        .AsNoTracking()
        .ToList();
}
````

你也可以在上下文实例级别改变默认的跟踪行为：

You can also change the default tracking behavior at the context instance level:

<!-- [!code-csharp[Main](samples/core/Querying/Querying/Tracking/Sample.cs?highlight=3)] -->
````csharp
using (var context = new BloggingContext())
{
    context.ChangeTracker.QueryTrackingBehavior = QueryTrackingBehavior.NoTracking;

    var blogs = context.Blogs.ToList();
}
````

> [!NOTE]
> 没有跟踪查询仍然执行身份解析。如果结果集多次包含同一实体，则这个实体类的实例将直接返回给结果集中的每个事件。然而，弱引用会被使用于继续跟踪先前返回的实体。如果同一标识的前一个结果超出了作用域，并且垃圾回收运行，您可能会得到一个新实体实例。有关更多信息，请参见[How Query Works](overview.html)。

> No tracking queries still perform identity resolution. If the result set contains the same entity multiple times, the same instance of the entity class will be returned for each occurrence in the result set. However, weak references are used to keep track of entities that have already been returned. If a previous result with the same identity goes out of scope, and garbage collection runs, you may get a new entity instance. For more information, see [How Query Works](overview.html).

## Tracking and projections

即使查询的结果类型不是实体类型，如果结果包含实体类型，它们仍将默认跟踪。在返回匿名类型的查询中，将跟踪结果集中`Blog`的实例。

Even if the result type of the query isn't an entity type, if the result contains entity types they will still be tracked by default. In the following query, which returns an anonymous type, the instances of `Blog` in the result set will be tracked.

<!-- [!code-csharp[Main](samples/core/Querying/Querying/Tracking/Sample.cs?highlight=7)] -->
````csharp
using (var context = new BloggingContext())
{
    var blog = context.Blogs
        .Select(b =>
            new
            {
                Blog = b,
                Posts = b.Posts.Count()
            });
}
````

如果结果集不包含任何实体类型，则不执行跟踪。在下面的查询中，返回一个匿名类型，其中有一些值来自实体（但没有实际实体类型的实例），不执行跟踪。

If the result set does not contain any entity types, then no tracking is performed. In the following query, which returns an anonymous type with some of the values from the entity (but no instances of the actual entity type), there is no tracking performed.

<!-- [!code-csharp[Main](samples/core/Querying/Querying/Tracking/Sample.cs)] -->
````csharp
using (var context = new BloggingContext())
{
    var blog = context.Blogs
        .Select(b =>
            new
            {
                Id = b.BlogId,
                Url = b.Url
            });
}
````
