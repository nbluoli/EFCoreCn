---
title: How Query Works | Microsoft Docs
author: rowanmiller
ms.author: divega
ms.date: 10/27/2016
ms.assetid: de2e34cd-659b-4cab-b5ed-7a979c6bf120
ms.technology: entity-framework-core
uid: core/querying/overview
---

# How Query Works

Entity Framework Core使用语言集成查询（LINQ）从数据库中查询数据。LINQ允许你使用C #（或选择你自己的.NET语言）写的强类型的查询的基础上派生的上下文和实体类。

Entity Framework Core uses Language Integrate Query (LINQ) to query data from the database. LINQ allows you to use C# (or your .NET language of choice) to write strongly typed queries based on your derived context and entity classes.

## The life of a query

以下是对每个查询过程的高级概述。

The following is a high level overview of the process each query goes through.

LINQ查询处理是通过Entity Framework Core建立一个表示数据库提供程序的预处理

1. The LINQ query is processed by Entity Framework Core to build a representation that is ready to be processed by the database provider
   
   结果是缓存的，所以每次执行查询时不需要执行此处理

   1. The result is cached so that this processing does not need to be done every time the query is executed

结果传递给数据库提供程序

2. The result is passed to the database provider
	
	数据库提供程序标识数据库中查询的哪个部分
  
    1. The database provider identifies which parts of the query can be evaluated in the database
  
    查询的这些部分被翻译成数据库特定的查询语言（例如关系数据库的SQL）
  
    2. These parts of the query are translated to database specific query language (e.g. SQL for a relational database)
   
    将一个或多个查询发送到数据库并返回结果集（结果是来自数据库的值，而不是实体实例）

    3. One or more queries are sent to the database and the result set returned (results are values from the database, not entity instances)

对于结果集中的每个项

3. For each item in the result set

	如果这是一个跟踪查询，EF检查数据表示的实体是否早已存在于上下文实例更改跟踪器中

   1. If this is a tracking query, EF checks if the data represents an entity already in the change tracker for the context instance

      * 如果是，则返回现有实体

	  * If so, the existing entity is returned

      * 如果没有，则创建一个新实体，设置更改跟踪，并返回新实体

	  * If not, a new entity is created, change tracking is setup, and the new entity is returned

   如果这是一个无跟踪查询，EF检查数据是否表示此查询的结果集中的实体

   2. If this is a no-tracking query, EF checks if the data represents an entity already in the result set for this query

      * 如果是，则返回现有实体 <sup>(1)</sup>

	  * If so, the existing entity is returned <sup>(1)</sup>

      * 如果没有，则创建并返回新实体

	  * If not, a new entity is created and returned

<sup>(1)</sup>没有跟踪查询使用弱引用来跟踪已经返回的实体。如果同一标识的前一个结果超出了作用域，并且垃圾回收运行，您可能会得到一个新实体实例。

<sup>(1)</sup> No tracking queries use weak references to keep track of entities that have already been returned. If a previous result with the same identity goes out of scope, and garbage collection runs, you may get a new entity instance.

## When queries are executed

当你调用LINQ运算符，你只是建立在内存上的表示查询。当结果被消耗时，查询才发送到数据库。

When you call LINQ operators, you are simply building up an in-memory representation of the query. The query is only sent to the database when the results are consumed.

查询结果发送到数据库中最常见操作是：

The most common operations that result in the query being sent to the database are:

* 在`for`循环中遍历结果

* Iterating the results in a `for` loop

使用操作如：`ToList`, `ToArray`, `Single`, `Count`

* Using an operator such as `ToList`, `ToArray`, `Single`, `Count`

绑定查询结果到UI

* Databinding the results of a query to a UI
