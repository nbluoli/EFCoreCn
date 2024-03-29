﻿---
title: Test components using Entity Framework | Microsoft Docs
author: rowanmiller
ms.author: divega

ms.date: 10/27/2016

ms.assetid: 1603be0c-69bc-4dd9-9a08-3d0129cdc6c1
ms.technology: entity-framework-core
 
uid: core/miscellaneous/testing/index
---

# Testing

You may want to test components using something that approximates connecting to the real database, without the overhead of actual database I/O operations.

There are two main options for doing this:
 * [SQLite in-memory mode](sqlite.md) allows you to write efficient tests against a provider that behaves like a relational database.
 * [The InMemory provider](in-memory.md) is a lightweight provider that has minimal dependencies, but does not always behave like a relational database.
