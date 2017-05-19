---
title: Including & Excluding Properties | Microsoft Docs
author: rowanmiller
ms.author: divega

ms.date: 10/27/2016

ms.assetid: e9dff604-3469-4a05-8f9e-18ac281d82a9
ms.technology: entity-framework-core
 
uid: core/modeling/included-properties
---
# Including & Excluding Properties

包含一个属性在模型中，意味着EF拥有这个属性的元数据，并将尝试从数据库读取和写入值。

Including a property in the model means that EF has metadata about that property and will attempt to read and write values from/to the database.

## Conventions

按照约定，公有属性有一个getter和setter将包含在模型中。

By convention, public properties with a getter and a setter will be included in the model.

## Data Annotations

你可能使用数据注释从模型中排除一个属性。

You can use Data Annotations to exclude a property from the model.

<!-- [!code-csharp[Main](samples/core/Modeling/DataAnnotations/Samples/IgnoreProperty.cs?highlight=6)] -->
````csharp
public class Blog
{
    public int BlogId { get; set; }
    public string Url { get; set; }

    [NotMapped]
    public DateTime LoadedFromDatabase { get; set; }
}
````

## Fluent API

你可以使用Fluent API从模型中排序一个属性。

You can use the Fluent API to exclude a property from the model.

<!-- [!code-csharp[Main](samples/core/Modeling/FluentAPI/Samples/IgnoreProperty.cs?highlight=7,8)] -->
````csharp
class MyContext : DbContext
{
    public DbSet<Blog> Blogs { get; set; }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Blog>()
            .Ignore(b => b.LoadedFromDatabase);
    }
}

public class Blog
{
    public int BlogId { get; set; }
    public string Url { get; set; }

    public DateTime LoadedFromDatabase { get; set; }
}
````
