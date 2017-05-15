---
title: Entity Framework Core | Microsoft Docs
author: rowanmiller
ms.author: divega

ms.date: 10/27/2016

ms.assetid: bc2a2676-bc46-493f-bf49-e3cc97994d57
ms.technology: entity-framework-core
 
uid: core/index
---

# Entity Framework Core

> [!NOTE]
> 本文档是关于EF Core。关于EF6.x 请看[Entity Framework 6](https://docs.microsoft.com/zh-cn/ef/ef6/index).

Entity Framework (EF) Core是一个轻量级的，可扩展的，跨平台版本的流行的实体框架数据访问技术。

Entity Framework (EF) Core is a lightweight, extensible, and cross-platform version of the popular Entity Framework data access technology.

EF Core是一个对象关系映射器（O/RM），使.NET开发人员能够使用.NET对象来处理数据库。它消除了大多数开发人员通常需要编写的数据访问代码的需要。EF Core支持许多数据库引擎，请参见[数据库提供者](/providers/index.html)以获取详细信息。

EF Core is an object-relational mapper (O/RM) that enables .NET developers to work with a database using .NET objects. It eliminates the need for most of the data-access code that developers usually need to write. EF Core supports many database engines, see [Database Providers](providers/index.md) for details.

如果你喜欢通过编写代码来学习，我们推荐您从[入门指南](get-started/index.html)学习 EF Core。

If you like to learn by writing code, we'd recommend one of our [Getting Started](get-started/index.md) guides to get you started with EF Core.

## Get Entity Framework Core

[Install the NuGet package](https://docs.nuget.org/ndocs/quickstart/use-a-package)请安装你想使用的database provider。查看[Database Providers](/providers/index.html)信息。

[Install the NuGet package](https://docs.nuget.org/ndocs/quickstart/use-a-package) for the database provider you want to use. See [Database Providers](providers/index.md) for information.

<!-- literal_block"ids  "dupnames  "names  "xml:space": "preserve", : "csharp",", "classes  "linenos": false, "backrefs  highlight_args} -->
````text
PM>  Install-Package Microsoft.EntityFrameworkCore.SqlServer
````

## The Model

EF Core和数据访问是使用同一个模型。模型由实体类和派生的上下文组成，它代表与数据库的会话，允许您查询和保存数据。查看[Creating a Model](modeling/index.html)来学习更多。

With EF Core, data access is performed using a model. A model is made up of entity classes and a derived context that represents a session with the database, allowing you to query and save data. See [Creating a Model](modeling/index.html) to learn more.

您可以从现有数据库生成一个模型，手工编码一个模型来匹配数据库，或者使用EF Migrations从模型中创建一个数据库（随着模型随时间变化而演化）。

You can generate a model from an existing database, hand code a model to match your database, or use EF Migrations to create a database from your model (and evolve it as your model changes over time).

<!-- literal_block"ids  "dupnames  "names  "xml:space": "preserve", : "csharp", "classes  "linenos": true, "backrefs  highlight_args} -->
````csharp
using Microsoft.EntityFrameworkCore;
using System.Collections.Generic;

namespace Intro
{
    public class BloggingContext : DbContext
    {
        public DbSet<Blog> Blogs { get; set; }
        public DbSet<Post> Posts { get; set; }

        protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
        {
            optionsBuilder.UseSqlServer(@"Server=(localdb)\mssqllocaldb;Database=MyDatabase;Trusted_Connection=True;");
        }
    }

    public class Blog
    {
        public int BlogId { get; set; }
        public string Url { get; set; }
        public int Rating { get; set; }
        public List<Post> Posts { get; set; }
    }

    public class Post
    {
        public int PostId { get; set; }
        public string Title { get; set; }
        public string Content { get; set; }

        public int BlogId { get; set; }
        public Blog Blog { get; set; }
    }
}
````

## Querying

你可以通过Linq从数据库获取实体类的实例。请查看[Querying Data](/querying/index.html)学习更多。

Instances of your entity classes are retrieved from the database using Language Integrated Query (LINQ). See [Querying Data](/querying/index.html) to learn more.

<!-- literal_block"ids  "dupnames  "names  "xml:space": "preserve", : "csharp", "classes  "linenos": true, "backrefs  highlight_args} -->
````csharp
using (var db = new BloggingContext())
{
    var blogs = db.Blogs
        .Where(b => b.Rating > 3)
        .OrderBy(b => b.Url)
        .ToList();
}
````

## Saving Data
通过实体类的实例可以创建、删除、修改数据库中的数据。请查看[Saving Data](/saving/index.html)学习更多。

Data is created, deleted, and modified in the database using instances of your entity classes. See [Saving Data](/saving/index.html) to learn more.

<!-- literal_block"ids  "dupnames  "names  "xml:space": "preserve", : "csharp", "classes  "linenos": true, "backrefs  highlight_args} -->
````csharp
using (var db = new BloggingContext())
{
    var blog = new Blog { Url = "http://sample.com" };
    db.Blogs.Add(blog);
    db.SaveChanges();
}
````
