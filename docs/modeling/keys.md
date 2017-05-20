---
title: Keys (primary) | Microsoft Docs
author: rowanmiller
ms.author: divega

ms.date: 10/27/2016

ms.assetid: 912ffef7-86a0-4cdc-a776-55f907459d20
ms.technology: entity-framework-core
 
uid: core/modeling/keys
---
# Keys (primary)

对于每个实体实例都有一个键服务作为主要唯一标识。当使用关系数据库时，它映射成一个*primary key*概念。你也可以配置唯一标识不映射成主键(参见 [Alternate Keys](alternate-keys.html)了解更多信息)。

A key serves as the primary unique identifier for each entity instance. When using a relational database this maps to the concept of a *primary key*. You can also configure a unique identifier that is not the primary key (see [Alternate Keys](alternate-keys.html) for more information).

## Conventions

按照约定，名为`Id`或`<type name>Id`的属性将被配置为实体的键。
By convention, a property named `Id` or `<type name>Id` will be configured as the key of an entity.

<!-- [!code-csharp[Main](samples/core/Modeling/Conventions/Samples/KeyId.cs?highlight=3)] -->
````csharp
class Car
{
    public string Id { get; set; }

    public string Make { get; set; }
    public string Model { get; set; }
}
````

<!-- [!code-csharp[Main](samples/core/Modeling/Conventions/Samples/KeyTypeNameId.cs?highlight=3)] -->
````csharp
class Car
{
    public string CarId { get; set; }

    public string Make { get; set; }
    public string Model { get; set; }
}
````

## Data Annotations

你可以使用数据注释配置单个属性成为实体的键。

You can use Data Annotations to configure a single property to be the key of an entity.

<!-- [!code-csharp[Main](samples/core/Modeling/DataAnnotations/Samples/KeySingle.cs?highlight=3,4)] -->
````csharp
class Car
{
    [Key]
    public string LicensePlate { get; set; }

    public string Make { get; set; }
    public string Model { get; set; }
}
````

## Fluent API

你也可以使用Fluent API去配置单个属性成为实体的键。

You can use the Fluent API to configure a single property to be the key of an entity.

<!-- [!code-csharp[Main](samples/core/Modeling/FluentAPI/Samples/KeySingle.cs?highlight=7,8)] -->
````csharp
class MyContext : DbContext
{
    public DbSet<Car> Cars { get; set; }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Car>()
            .HasKey(c => c.LicensePlate);
    }
}

class Car
{
    public string LicensePlate { get; set; }

    public string Make { get; set; }
    public string Model { get; set; }
}
````
你也可以使用Fluent API配置多个属性成为实体的键（即复合键）。复合键只能通过Fluent API配置——约定永远不会设置复合键并且数据注释也不能配置复合键。
You can also use the Fluent API to configure multiple properties to be the key of an entity (known as a composite key). Composite keys can only be configured using the Fluent API - conventions will never setup a composite key and you can not use Data Annotations to configure one.

<!-- [!code-csharp[Main](samples/core/Modeling/FluentAPI/Samples/KeyComposite.cs?highlight=7,8)] -->
````csharp
class MyContext : DbContext
{
    public DbSet<Car> Cars { get; set; }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Car>()
            .HasKey(c => new { c.State, c.LicensePlate });
    }
}

class Car
{
    public string State { get; set; }
    public string LicensePlate { get; set; }

    public string Make { get; set; }
    public string Model { get; set; }
}
````
