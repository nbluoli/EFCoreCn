---
title: Including & Excluding Types | Microsoft Docs
author: rowanmiller
ms.author: divega

ms.date: 10/27/2016

ms.assetid: cbe6935e-2679-4b77-8914-a8d772240cf1
ms.technology: entity-framework-core
 
uid: core/modeling/included-types
---
# 包含与排除类型
# Including & Excluding Types

在模型中包含一个类型意味着EF拥有这个类型的元数据并将尝试从数据库读取和写入这个类型的实例。

Including a type in the model means that EF has metadata about that type and will attempt to read and write instances from/to the database.

##约定
## Conventions

按照约定，通过上下文`DbSet`属性暴露的类型是被包含在模型中的。此外，在`OnModelCreating`方法中提到的类型也是被包含在模型中的。最后，任何通过导航属性递归扫描发现的类型，也包含在模型中。

By convention, types that are exposed in `DbSet` properties on your context are included in your model. In addition, types that are mentioned in the `OnModelCreating` method are also included. Finally, any types that are found by recursively exploring the navigation properties of discovered types are also included in the model.

** 例如，在下面的代码列表中所发现三种类型：**

** For example, in the following code listing all three types are discovered:**

* `Blog`因为它暴露在上下文`DbSet`属性中

* `Blog` because it is exposed in a `DbSet` property on the context

* `Post` 因为它通过`Blog.Posts`导航属性被发现

* `Post` because it is discovered via the `Blog.Posts` navigation property

* `AuditEntry`因为它在`OnModelCreating`中被提到。

* `AuditEntry` because it is mentioned in `OnModelCreating`

<!-- [!code-csharp[Main](samples/core/Modeling/Conventions/Samples/IncludedTypes.cs?highlight=3,7,16)] -->
````csharp
class MyContext : DbContext
{
    public DbSet<Blog> Blogs { get; set; }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<AuditEntry>();
    }
}

public class Blog
{
    public int BlogId { get; set; }
    public string Url { get; set; }

    public List<Post> Posts { get; set; }
}

public class Post
{
    public int PostId { get; set; }
    public string Title { get; set; }
    public string Content { get; set; }

    public Blog Blog { get; set; }
}

public class AuditEntry
{
    public int AuditEntryId { get; set; }
    public string Username { get; set; }
    public string Action { get; set; }
}
````
## 数据注释
## Data Annotations

你可以使用数据注释将一个类型从模型中排除。

You can use Data Annotations to exclude a type from the model.

<!-- [!code-csharp[Main](samples/core/Modeling/DataAnnotations/Samples/IgnoreType.cs?highlight=9)] -->
````csharp
public class Blog
{
    public int BlogId { get; set; }
    public string Url { get; set; }

    public BlogMetadata Metadata { get; set; }
}

[NotMapped]
public class BlogMetadata
{
    public DateTime LoadedFromDatabase { get; set; }
}
````

## Fluent API

你可以使用Fluent API将一个类型从模型中排除。

You can use the Fluent API to exclude a type from the model.

<!-- [!code-csharp[Main](samples/core/Modeling/FluentAPI/Samples/IgnoreType.cs?highlight=7)] -->
````csharp
class MyContext : DbContext
{
    public DbSet<Blog> Blogs { get; set; }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Ignore<BlogMetadata>();
    }
}

public class Blog
{
    public int BlogId { get; set; }
    public string Url { get; set; }

    public BlogMetadata Metadata { get; set; }
}

public class BlogMetadata
{
    public DateTime LoadedFromDatabase { get; set; }
}
````
