---
title: Querying Data | Microsoft Docs
author: rowanmiller
ms.author: divega

ms.date: 10/27/2016

ms.assetid: 7c65ec3e-46c8-48f8-8232-9e31f96c277b
ms.technology: entity-framework-core
 
uid: core/querying/index
---
# Querying Data

Entity Framework Core使用Language Integrate Query (LINQ)从数据库中查询数据。LINQ允许你使用 C# (或者选择您的.NET 语言) 写的强类型查询基于派生上下文和实体类。LINQ查询的表示传递给数据库提供程序，被翻译成数据库特定查询语言（如SQL之于关系数据库）。有关如何处理查询的详细信息，请参见 [How Query Works](overview.html)。

Entity Framework Core uses Language Integrate Query (LINQ) to query data from the database. LINQ allows you to use C# (or your .NET language of choice) to write strongly typed queries based on your derived context and entity classes. A representation of the LINQ query is passed to the database provider, to be translated in database-specific query language (e.g. SQL for a relational database). For more detailed information on how a query is processed, see [How Query Works](overview.html).
