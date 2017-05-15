﻿---
title: Memory-Optimized Tables | Microsoft Docs
author: rowanmiller
ms.author: divega

ms.date: 10/27/2016

ms.assetid: 2e007c82-c6e4-45bb-8129-851b79ec1a0a
ms.technology: entity-framework-core
 
uid: core/providers/sql-server/memory-optimized-tables
---

# Memory-Optimized Tables

> [!NOTE]
> This documentation is for EF Core. For EF6.x, see [Entity Framework 6](../../../ef6/index.md).

[Memory-Optimized Tables](https://msdn.microsoft.com/en-us/library/dn133165.aspx) are a feature of SQL Server where the entire table resides in memory. A second copy of the table data is maintained on disk, but only for durability purposes. Data in memory-optimized tables is only read from disk during database recovery. For example, after a server restart.

> [!NOTE]
> Memory-Optimized Table support was introduced in EF Core 1.1.0. If you are using an earlier release, the functionality shown in this article will not be available.

## Configuring a memory-optimized table

You can specify that the table an entity is mapped to is memory-optimized. When using EF Core to create and maintain a database based on your model (either with migrations or `Database.EnsureCreated()`), a memory-optimized table will be created for these entities.

```c#
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Blog>()
        .ForSqlServerIsMemoryOptimized();
}
```
