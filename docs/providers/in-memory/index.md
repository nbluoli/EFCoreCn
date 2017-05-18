---
title: InMemory | Microsoft Docs
author: rowanmiller
ms.author: divega

ms.date: 10/27/2016

ms.assetid: 9af0cba7-7605-4f8f-9cfa-dd616fcb880c
ms.technology: entity-framework-core

uid: core/providers/in-memory/index
---
# InMemory

数据库提供程序允许Entity Framework Core与内存数据库一起使用。这是非常有用的，当测试代码使用Entity Framework Core时。提供程序作为[EntityFramework GitHub project](https://github.com/aspnet/EntityFramework)的一部分维护。

This database provider allows Entity Framework Core to be used with an in-memory database. This is useful when testing code that uses Entity Framework Core. The provider is maintained as part of the [EntityFramework GitHub project](https://github.com/aspnet/EntityFramework).

## Install

Install the [Microsoft.EntityFrameworkCore.InMemory NuGet package](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.InMemory/).

<!-- literal_block"ids  "classes  "xml:space": "preserve", "backrefs  "linenos": false, "dupnames  : "csharp",", highlight_args}, "names": [] -->
````text

   PM>  Install-Package Microsoft.EntityFrameworkCore.InMemory
````

## Get Started

下列资源将帮助你启动此提供程序。

The following resources will help you get started with this provider.
* [Testing with InMemory](../../miscellaneous/testing/in-memory.html)

* [UnicornStore Sample Application Tests](https://github.com/rowanmiller/UnicornStore/blob/master/UnicornStore/src/UnicornStore.Tests/Controllers/ShippingControllerTests.cs)

## Supported Database Engines

* Built-in in-memory database (designed for testing purposes only)

## Supported Platforms

* .NET Framework (4.5.1 onwards)

* .NET Core

* Mono (4.2.0 onwards)

* Universal Windows Platform
