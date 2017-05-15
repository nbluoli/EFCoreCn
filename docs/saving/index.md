---
title: Saving Data | Microsoft Docs
author: rowanmiller
ms.author: divega

ms.date: 10/27/2016

ms.assetid: ef044629-feca-4fd1-a48f-d208daedaf92
ms.technology: entity-framework-core
 
uid: core/saving/index
---
# Saving Data

每个上下文实例都有一个`ChangeTracker`，负责跟踪需要写入数据库的变化。当你改变你的实体类的实例，这些变化都记录在`ChangeTracker`然后写入数据库，当你调用`SaveChanges`。数据库提供程序负责将更改转换为数据库特定操作（如关系数据库的`INSERT`、`UPDATE`和`DELETE`命令）。

Each context instance has a `ChangeTracker` that is responsible for keeping track of changes that need to be written to the database. As you make changes to instances of your entity classes, these changes are recorded in the `ChangeTracker` and then written to the database when you call `SaveChanges`. The database provider is responsible for translating the changes into database-specific operations (e.g. `INSERT`, `UPDATE`, and `DELETE` commands for a relational database).
