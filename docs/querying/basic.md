---
title: Basic Query | Microsoft Docs
author: rowanmiller
ms.author: divega

ms.date: 10/27/2016

ms.assetid: ab6e35f1-397f-41c0-9ef4-85aec5466377
ms.technology: entity-framework-core
 
uid: core/querying/basic
---
# Basic Query

学会如何使用Language Integrate Query (LINQ)从数据库中加载实体。

Learn how to load entities from the database using Language Integrate Query (LINQ).

> [!TIP]
> 你可在GitHub上查看本文的[示例项目](https://github.com/aspnet/EntityFramework.Docs/tree/master/samples/core/Querying)。
> You can view this article's [sample](https://github.com/aspnet/EntityFramework.Docs/tree/master/samples/core/Querying) on GitHub.

## 101 LINQ samples

本页展示了几个实现Entity Framework Core的通用任务示例。关于一些常见可能用到LINQ的示例，参见[101 LINQ Samples](https://code.msdn.microsoft.com/101-LINQ-Samples-3fb9811b)。

This page shows a few examples to achieve common tasks with Entity Framework Core. For an extensive set of samples showing what is possible with LINQ, see [101 LINQ Samples](https://code.msdn.microsoft.com/101-LINQ-Samples-3fb9811b).

## Loading all data

<!-- [!code-csharp[Main](samples/core/Querying/Querying/Basics/Sample.cs)] -->
````csharp
using (var context = new BloggingContext())
{
    var blogs = context.Blogs.ToList();
}
````

## Loading a single entity

<!-- [!code-csharp[Main](samples/core/Querying/Querying/Basics/Sample.cs)] -->
````csharp
using (var context = new BloggingContext())
{
    var blog = context.Blogs
        .Single(b => b.BlogId == 1);
}
````

## Filtering

<!-- [!code-csharp[Main](samples/core/Querying/Querying/Basics/Sample.cs)] -->
````csharp
using (var context = new BloggingContext())
{
    var blogs = context.Blogs
        .Where(b => b.Url.Contains("dotnet"))
        .ToList();
}
````