---
title: Raw SQL Queries | Microsoft Docs
author: rowanmiller
ms.author: divega

ms.date: 10/27/2016

ms.assetid: 70aae9b5-8743-4557-9c5d-239f688bf418
ms.technology: entity-framework-core
 
uid: core/querying/raw-sql
---
# 原生SQL查询
# Raw SQL Queries

Entity Framework Core允许您在使用关系数据库时对原始sql查询进行拉取。如果你想执行的查询不能用LINQ表达，这是非常有用的，或者如果使用LINQ查询导致低效的SQL发送到数据库。

Entity Framework Core allows you to drop down to raw SQL queries when working with a relational database. This can be useful if the query you want to perform can't be expressed using LINQ, or if using a LINQ query is resulting in inefficient SQL being sent to the database.

> [!TIP]
> 你可以在GitHub上查看本文的【示例项目】(https://github.com/aspnet/EntityFramework.Docs/tree/master/samples/core/Querying)。
> You can view this article's [sample](https://github.com/aspnet/EntityFramework.Docs/tree/master/samples/core/Querying) on GitHub.

## 限制
## Limitations

使用原生SQL查询需要注意以下几个限制：

There are a couple of limitations to be aware of when using raw SQL queries:

* SQL查询只能用于返回属于模型部分的实体类型。在我们的待定项中会增强[enable returning ad-hoc types from raw SQL queries](https://github.com/aspnet/EntityFramework/issues/1862)。
* SQL queries can only be used to return entity types that are part of your model. There is an enhancement on our backlog to [enable returning ad-hoc types from raw SQL queries](https://github.com/aspnet/EntityFramework/issues/1862).

* SQL查询必须返回实体类型的所有属性。
* The SQL query must return data for all properties of the entity type.

* 结果集中的列名必须匹配属性映射到的列名。注意，这不同于EF 6.x在属性/列映射为原生的SQL查询和结果集的列名称忽略对属性名相匹配。
* The column names in the result set must match the column names that properties are mapped to. Note this is different from EF6.x where property/column mapping was ignored for raw SQL queries and result set column names had to match the property names.

 SQL查询不能包含相关数据。然而，在许多情况下，你可以在上面使用原生SQL查询，然后使用`Include`操作符返回相关的数据（参见[Including related data](#including-related-data)小节）。
* The SQL query cannot contain related data. However, in many cases you can compose on top of the query using the `Include` operator to return related data (see [Including related data](#including-related-data)).

## Basic raw SQL queries
你可以使用*FromSql*扩展方法开始一个基于原生SQL查询的LINQ查询。

You can use the *FromSql* extension method to begin a LINQ query based on a raw SQL query.

<!-- [!code-csharp[Main](samples/core/Querying/Querying/RawSQL/Sample.cs)] -->
````csharp
var blogs = context.Blogs
    .FromSql("SELECT * FROM dbo.Blogs")
    .ToList();
````
原生SQL查询可以用于执行存储过程。

Raw SQL queries can be used to execute a stored procedure.

<!-- [!code-csharp[Main](samples/core/Querying/Querying/RawSQL/Sample.cs)] -->
````csharp
var blogs = context.Blogs
    .FromSql("EXECUTE dbo.GetMostPopularBlogs")
    .ToList();
````

## Passing parameters

正如任何API接受SQL一样，最重要的是参数化任何用户输入以防止SQL注入攻击。你可以在SQL查询字符串包含参数占位符，然后提供参数值作为额外的参数。任何参数值，你将自动转换为`DbParameter`。

As with any API that accepts SQL, it is important to parameterize any user input to protect against a SQL injection attack. You can include parameter placeholders in the SQL query string and then supply parameter values as additional arguments. Any parameter values you supply will automatically be converted to a `DbParameter`.

下面的示例将单个参数传递给存储过程。虽然这可能看起来像`String.Format`语法，所提供的值被封装在一个参数中，并且生成的参数名插入了`{0}`占位符指定的位置。

The following example passes a single parameter to a stored procedure. While this may look like `String.Format` syntax, the supplied value is wrapped in a parameter and the generated parameter name inserted where the `{0}` placeholder was specified.

<!-- [!code-csharp[Main](samples/core/Querying/Querying/RawSQL/Sample.cs)] -->
````csharp
var user = "johndoe";

var blogs = context.Blogs
    .FromSql("EXECUTE dbo.GetMostPopularBlogsForUser {0}", user)
    .ToList();
````

你也可以建立一个DbParameter提供它作为一个参数值。允许在SQL查询字符串中使用命名参数

You can also construct a DbParameter and supply it as a parameter value. This allows you to use named parameters in the SQL query string

<!-- [!code-csharp[Main](samples/core/Querying/Querying/RawSQL/Sample.cs)] -->
````csharp
var user = new SqlParameter("user", "johndoe");

var blogs = context.Blogs
    .FromSql("EXECUTE dbo.GetMostPopularBlogsForUser @user", user)
    .ToList();
````

## Composing with LINQ

如果在数据库中组装SQL查询，然后你可以在顶部的初始化原生SQL查询使用LINQ运算符组成。SQL查询，可以与`SELECT`关键字组成。

If the SQL query can be composed on in the database, then you can compose on top of the initial raw SQL query using LINQ operators. SQL queries that can be composed on being with the `SELECT` keyword.

下面的示例使用一个SQL查询，选择从一个Table-Valued Function (TVF)然后结合LINQ进行过滤和排序。

The following example uses a raw SQL query that selects from a Table-Valued Function (TVF) and then composes on it using LINQ to perform filtering and sorting.

<!-- [!code-csharp[Main](samples/core/Querying/Querying/RawSQL/Sample.cs)] -->
````csharp
var searchTerm = ".NET";

var blogs = context.Blogs
    .FromSql("SELECT * FROM dbo.SearchBlogs {0}", searchTerm)
    .Where(b => b.Rating > 3)
    .OrderByDescending(b => b.Rating)
    .ToList();
````

### Including related data

在原生SQL查询中结合LINQ操作符可以包含相关数据。

Composing with LINQ operators can be used to include related data in the query.

<!-- [!code-csharp[Main](samples/core/Querying/Querying/RawSQL/Sample.cs)] -->
````csharp
var searchTerm = ".NET";
    
var blogs = context.Blogs
    .FromSql("SELECT * FROM dbo.SearchBlogs {0}", searchTerm)
    .Include(b => b.Posts)
    .ToList();
````
 