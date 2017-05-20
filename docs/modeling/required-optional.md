---
title: Required/optional properties | Microsoft Docs
author: rowanmiller
ms.author: divega

ms.date: 10/27/2016

ms.assetid: ddaa0a54-9f43-4c34-aae3-f95c96c69842
ms.technology: entity-framework-core
 
uid: core/modeling/required-optional
---
# 必须的/可选的属性
# Required/optional properties

一个属性被认为是可选的，那么包含`null`对于它是有效的。如果一个属性被赋值为`null`是无效的，则它被认为是必须的属性。

A property is considered optional if it is valid for it to contain `null`. If `null` is not a valid value to be assigned to a property then it is considered to be a required property.

## Conventions

按照约定，CLR类型可以包含NULL的属性将被配置为可选（`string`, `int?`, `byte[]`等）。属性的CLR类型不能包含null将被配置为必须的（`int`, `decimal`, `bool`等）。

By convention, a property whose CLR type can contain null will be configured as optional (`string`, `int?`, `byte[]`, etc.). Properties whose CLR type cannot contain null will be configured as required (`int`, `decimal`, `bool`, etc.).

> [!NOTE]
> CLR类型不能包含null的属性不能被配置为可选属性。属性将始终被Entity Framework认为是必须的。
> A property whose CLR type cannot contain null cannot be configured as optional. The property will always be considered required by Entity Framework.

## Data Annotations

可以使用数据注释来指明是必须属性。

You can use Data Annotations to indicate that a property is required.

<!-- [!code-csharp[Main](samples/core/Modeling/DataAnnotations/Samples/Required.cs?highlight=4)] -->
````csharp
public class Blog
{
    public int BlogId { get; set; }
    [Required]
    public string Url { get; set; }
}
````

## Fluent API

可以使用Fluent API指明一个属性是必须的。

You can use the Fluent API to indicate that a property is required.

<!-- [!code-csharp[Main](samples/core/Modeling/FluentAPI/Samples/Required.cs?highlight=7,8,9)] -->
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

public class Blog
{
    public int BlogId { get; set; }
    public string Url { get; set; }
}
````
