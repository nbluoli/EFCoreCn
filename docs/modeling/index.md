---
title: Creating a Model | Microsoft Docs
author: rowanmiller
ms.author: divega

ms.date: 10/27/2016

ms.assetid: 88253ff3-174e-485c-b3f8-768243d01ee1
ms.technology: entity-framework-core
 
uid: core/modeling/index
---
# Creating a Model

> [!NOTE]
> This documentation is for EF Core. For EF6.x, see [Entity Framework 6](../../ef6/index.md).

Entity Framework使用一组约定，建立基于实体类模型的模式。您可以指定额外的配置以补充和/或重写所发现约定。

Entity Framework uses a set of conventions to build a model based on the shape of your entity classes. You can specify additional configuration to supplement and/or override what was discovered by convention.

本文介绍的配置，模型可以应用到任何数据存储和关系数据库。提供程序还可以启用针对于特定的数据存储配置。关于提供程序特定的配置文件请看[Database Providers](../providers/index.html)节。

This article covers configuration that can be applied to a model targeting any data store and that which can be applied when targeting any relational database. Providers may also enable configuration that is specific to a particular data store. For documentation on provider specific configuration see the [Database Providers](../providers/index.html) section.

> [!TIP]
> You can view this article’s [sample](https://github.com/aspnet/EntityFramework.Docs/tree/master/samples) on GitHub.

## Methods of configuration

### Fluent API

你可以在你的派生上下文中重写`OnModelCreating`，使用`ModelBuilder API`去配置你的模型。这是最流行的配置方法，允许在不修改实体类的情况指定配置。Fluent API配置优先级最高，它可以覆盖约定和数据注释。

You can override the `OnModelCreating` method in your derived context and use the `ModelBuilder API` to configure your model. This is the most powerful method of configuration and allows configuration to be specified without modifying your entity classes. Fluent API configuration has the highest precedence and will override conventions and data annotations.

<!-- [!code-csharp[Main](samples/core/Modeling/FluentAPI/Samples/Required.cs?range=5-15&highlight=5-10)] -->

````csharp
    class MyContext : DbContext
    {
        public DbSet<Blog> Blogs { get; set; }

        protected override void OnModelCreating(ModelBuilder modelBuilder)
        {
            modelBuilder.Entity<Blog>()
                .Property(b => b.Url)
                .IsRequired();
        }
    }
````

### Data Annotations

你也可以应用attributes（也就是数据注释）在类和properties上。数据注释将覆盖约定，但会被Fluent API配置覆盖。

You can also apply attributes (known as Data Annotations) to your classes and properties. Data annotations will override conventions, but will be overwritten by Fluent API configuration.

<!-- [!code-csharp[Main](samples/core/Modeling/DataAnnotations/Samples/Required.cs?range=11-16&highlight=4)] -->

````csharp
    public class Blog
    {
        public int BlogId { get; set; }
        [Required]
        public string Url { get; set; }
    }
````
