---
title: Loading Related Data | Microsoft Docs
author: rowanmiller
ms.author: divega
ms.date: 10/27/2016
ms.assetid: f9fb64e2-6699-4d70-a773-592918c04c19
ms.technology: entity-framework-core
uid: core/querying/related-data
---
#加载相关数据
# Loading Related Data

Entity Framework Core允许您使用模型中的导航属性加载相关实体。有三个常见的O/RM模式用于加载相关数据。

Entity Framework Core allows you to use the navigation properties in your model to load related entities. There are three common O/RM patterns used to load related data.

* ** 立即加载** 意味着相关的数据从数据库加载作为初始查询的一部分。

* **Eager loading** means that the related data is loaded from the database as part of the initial query.

* **显式加载** 意味着相关数据在稍后的时间从数据库中显式加载。

* **Explicit loading** means that the related data is explicitly loaded from the database at a later time.

* **懒加载** 意味着当访问导航属性时，相关数据才从数据库加载。EF Core还不能懒加载。

* **Lazy loading** means that the related data is transparently loaded from the database when the navigation property is accessed. Lazy loading is not yet possible with EF Core.

> [!TIP]
> 你可以GitHub查看文本的[示例项目](https://github.com/aspnet/EntityFramework.Docs/tree/master/samples/core/Querying)。

> You can view this article's [sample](https://github.com/aspnet/EntityFramework.Docs/tree/master/samples/core/Querying) on GitHub.

## Eager loading

可以使用`Include`方法指定要包含在查询结果中的相关数据。在下面的示例中，返回的blogs结果中将有与相关posts填充的`Posts`属性。

You can use the `Include` method to specify related data to be included in query results. In the following example, the blogs that are returned in the results will have their `Posts` property populated with the related posts.

<!--[!code-csharp[Main](../../../samples/core/Querying/Querying/RelatedData/Sample.cs#SingleInclude)]-->

```csharp
using (var context = new BloggingContext())
{
    var blogs = context.Blogs
        .Include(blog => blog.Posts)
        .ToList();
}
```

> [!TIP]

> Entity Framework Core将自动将导航属性设置为以前加载到上下文实例中的任何其他实体。因此，即使您没有明确地将导航属性数据包含进来，但如果一些或所有相关实体都已加载，则属性仍可能填充。

> Entity Framework Core will automatically fix-up navigation properties to any other entities that were previously loaded into the context instance. So even if you don't explicitly include the data for a navigation property, the property may still be populated if some or all of the related entities were previously loaded.

可以在单个查询中包含来自多个关系的相关数据。

You can include related data from multiple relationships in a single query.

<!--[!code-csharp[Main](../../../samples/core/Querying/Querying/RelatedData/Sample.cs#MultipleIncludes)]-->

```csharp
using (var context = new BloggingContext())
{
    var blogs = context.Blogs
        .Include(blog => blog.Posts)
        .Include(blog => blog.Owner)
        .ToList();
}
```

### Including multiple levels

使用`ThenInclude`方法，你可以一直往下关联包含多级相关数据。下面的示例加载所有blogs，以及相关posts和每个post的作者。

You can drill down thru relationships to include multiple levels of related data using the `ThenInclude` method. The following example loads all blogs, their related posts, and the author of each post.

<!--[!code-csharp[Main](../../../samples/core/Querying/Querying/RelatedData/Sample.cs#SingleThenInclude)]-->
```csharp
using (var context = new BloggingContext())
{
    var blogs = context.Blogs
        .Include(blog => blog.Posts)
            .ThenInclude(post => post.Author)
        .ToList();
}
```
你可以连续多次调用`ThenInclude`持续更进一步的将相关数据包括进来。

You can chain multiple calls to `ThenInclude` to continue including further levels of related data.

<!--[!code-csharp[Main](../../../samples/core/Querying/Querying/RelatedData/Sample.cs#MultipleThenIncludes)]-->

```csharp
using (var context = new BloggingContext())
{
    var blogs = context.Blogs
        .Include(blog => blog.Posts)
            .ThenInclude(post => post.Author)
                .ThenInclude(author => author.Photo)
        .ToList();
}
```

您可以将来自多个级别和多个根的同一查询包含进来的所有的相关数据合并。

You can combine all of this to include related data from multiple levels and multiple roots in the same query.

<!--[!code-csharp[Main](../../../samples/core/Querying/Querying/RelatedData/Sample.cs#IncludeTree)]-->

```csharp
using (var context = new BloggingContext())
{
    var blogs = context.Blogs
        .Include(blog => blog.Posts)
            .ThenInclude(post => post.Author)
            .ThenInclude(author => author.Photo)
        .Include(blog => blog.Owner)
            .ThenInclude(owner => owner.Photo)
        .ToList();
}
```

您可能想要包含其中一个实体的多个相关实体。例如，当查询`Blog`的时候，你可能想要包括`Posts`，然后以及包括`Posts`的`Author`和`Tags`。为此，需要指定从根开始的每个包含路径。例如，`Blog -> Posts -> Author`和 `Blog -> Posts -> Tags`。这并不意味着你会得到冗余的联结，在大多数情况下EF生成SQL时将加强联结。

You may want to include multiple related entities for one of the entities that is being included. For example, when querying `Blog`s, you include `Posts` and then want to include both the `Author` and `Tags` of the `Posts`. To do this, you need to specify each include path starting at the root. For example, `Blog -> Posts -> Author` and `Blog -> Posts -> Tags`. This does not mean you will get redundant joins, in most cases EF will consolidate the joins when generating SQL.

<!--[!code-csharp[Main](../../../samples/core/Querying/Querying/RelatedData/Sample.cs#MultipleLeafIncludes)]-->

```csharp
using (var context = new BloggingContext())
{
    var blogs = context.Blogs
        .Include(blog => blog.Posts)
            .ThenInclude(post => post.Author)
        .Include(blog => blog.Posts)
            .ThenInclude(post => post.Tags)
        .ToList();
}
```

### Ignored includes

如果更改查询，使其不再返回查询开始的实体类型的实例，则包含运算符会被忽略。

If you change the query so that it no longer returns instances of the entity type that the query began with, then the include operators are ignored.

在下面的示例中，包含运算符是基于`Blog`的，但`Select`运算符用于更改查询以返回匿名类型。在这种情况下，包含操作符不受影响。

In the following example, the include operators are based on the `Blog`, but then the `Select` operator is used to change the query to return an anonymous type. In this case, the include operators have no effect.

<!--[!code-csharp[Main](../../../samples/core/Querying/Querying/RelatedData/Sample.cs#IgnoredInclude)]-->

```csharp
using (var context = new BloggingContext())
{
    var blogs = context.Blogs
        .Include(blog => blog.Posts)
        .Select(blog => new
        {
            Id = blog.BlogId,
            Url = blog.Url
        })
        .ToList();
}
```

默认情况下， EF Core将在包含操作被忽略时记录一个警告。参看[Logging](../miscellaneous/logging.html)查看日志输出的更多信息。当包含操作符被忽略或抛出或不做任何操作时，您可以更改该行为。你可以这样做，在你的自己的上下文选项中设置——通常是在`DbContext.OnConfiguring`中，你也可以在 ASP.NET Core的`Startup.cs`中设置。

By default, EF Core will log a warning when include operators are ignored. See [Logging](../miscellaneous/logging.html) for more information on viewing logging output. You can change the behavior when an include operator is ignored to either throw or do nothing. This is done when setting up the options for your context - typically in `DbContext.OnConfiguring`, or in `Startup.cs` if you are using ASP.NET Core.

<!--[!code-csharp[Main](../../../samples/core/Querying/Querying/RelatedData/ThrowOnIgnoredInclude/BloggingContext.cs#OnConfiguring)]-->

```csharp
protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
{
    optionsBuilder
        .UseSqlServer(@"Server=(localdb)\mssqllocaldb;Database=EFQuerying;Trusted_Connection=True;")
        .ConfigureWarnings(warnings => warnings.Throw(CoreEventId.IncludeIgnoredWarning));
}
```

## Explicit loading

> [!NOTE]
> 显式加载最早是从EF Core 1.1.0开始支持。如果使用较早版本，本节中显示加载功能将不可用。

> Explicit Loading support was introduced in EF Core 1.1.0. If you are using an earlier release, the functionality shown in this section will not be available.

你可以通过`DbContext.Entry(...)API显示加载一个导航属性`。

You can explicitly load a navigation property via the `DbContext.Entry(...)` API.

<!--[!code-csharp[Main](../../../samples/core/Querying/Querying/RelatedData/Sample.cs#Eager)]-->

```csharp
using (var context = new BloggingContext())
{
    var blog = context.Blogs
        .Single(b => b.BlogId == 1);

    context.Entry(blog)
        .Collection(b => b.Posts)
        .Load();

    context.Entry(blog)
        .Reference(b => b.Owner)
        .Load();
}
```

### Querying related entities

你也可通过一个LINQ查询获取导航属性的内容。

You can also get a LINQ query that represents the contents of a navigation property.

这允许您做一些事情，例如在相关实体上运行聚合运算符而不将它们加载到内存中。

This allows you to do things such as running an aggregate operator over the related entities without loading them into memory.

<!--[!code-csharp[Main](../../../samples/core/Querying/Querying/RelatedData/Sample.cs#NavQueryAggregate)]-->

```csharp
using (var context = new BloggingContext())
{
    var blog = context.Blogs
        .Single(b => b.BlogId == 1);

    var postCount = context.Entry(blog)
        .Collection(b => b.Posts)
        .Query()
        .Count();
}
```
还可以将相关实体加载到内存中进行筛选。

You can also filter which related entities are loaded into memory.

<!--[!code-csharp[Main](../../../samples/core/Querying/Querying/RelatedData/Sample.cs#NavQueryFiltered)]-->

```csharp
using (var context = new BloggingContext())
{
    var blog = context.Blogs
        .Single(b => b.BlogId == 1);

    var goodPosts = context.Entry(blog)
        .Collection(b => b.Posts)
        .Query()
        .Where(p => p.Rating > 3)
        .ToList();
}
```

## Lazy loading

EF Core暂不是支持懒加载。你可以查看[lazy loading item on our backlog](https://github.com/aspnet/EntityFramework/issues/3797)了解最新进展。

Lazy loading is not yet supported by EF Core. You can view the [lazy loading item on our backlog](https://github.com/aspnet/EntityFramework/issues/3797) to track this feature.

## Related data and serialization

因为EF Core会自动修复导航属性，你可以在你的对象图中结束循环。例如，加载一个blog及其相关的posts会导致一个blog对象引用一个posts的集合。每个posts又引用一个blog。

Because EF Core will automatically fix-up navigation properties, you can end up with cycles in your object graph. For example, Loading a blog and it's related posts will result in a blog object that references a collection of posts. Each of those posts will have a reference back to the blog.

一些序列化框架不允许这样的循环。例如，如果一个Json.NET遇到一个循环将抛出以下异常。

Some serialization frameworks do not allow such cycles. For example, Json.NET will throw the following exception if a cycle is encoutered.

> Newtonsoft.Json.JsonSerializationException: Self referencing loop detected for property 'Blog' with type 'MyApplication.Models.Blog'.

如果您使用的是ASP.NET Core，在对象图中发现循环你可以配置Json.NET忽略。在`Startup.cs`中的`ConfigureServices(...)`方法这样做。

If you are using ASP.NET Core, you can configure Json.NET to ignore cycles that it finds in the object graph. This is done in the `ConfigureServices(...)` method in `Startup.cs`.

```charp
public void ConfigureServices(IServiceCollection services)
{
    ...

    services.AddMvc()
        .AddJsonOptions(
            options => options.SerializerSettings.ReferenceLoopHandling = Newtonsoft.Json.ReferenceLoopHandling.Ignore
        );

    ...
}
```