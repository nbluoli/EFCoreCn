---
title: Basic Save | Microsoft Docs
author: rowanmiller
ms.author: divega

ms.date: 10/27/2016

ms.assetid: 850d842e-3fad-4ef2-be17-053768e97b9e
ms.technology: entity-framework-core
 
uid: core/saving/basic
---
# Basic Save

学习如何使用上下文和实体类来添加、修改和移除数据。

Learn how to add, modify, and remove data using your context and entity classes.

> [!提示]
> 你可以在GitHub查看这篇文章的[示例代码](https://github.com/aspnet/EntityFramework.Docs/tree/master/samples/core/Saving/Saving/Basics/)

## Adding Data

使用*DbSet.Add*方法去添加一个实体类的新实例。当你调用*SaveChanges*方法时，数据将被插入到数据库中。

Use the *DbSet.Add* method to add new instances of your entity classes. The data will be inserted in the database when you call *SaveChanges*.

<!-- [!code-csharp[Main](samples/core/Saving/Saving/Basics/Sample.cs)] -->
````csharp
        using (var db = new BloggingContext())
        {
            var blog = new Blog { Url = "http://sample.com" };
            db.Blogs.Add(blog);
            db.SaveChanges();

            Console.WriteLine(blog.BlogId + ": " +  blog.Url);
        }
````

## Updating Data

EF将自动检测到由上下文跟踪的现有实体所做的更改。这包括从数据库加载/查询的实体，以及以前添加并保存到数据库的实体。

EF will automatically detect changes made to an existing entity that is tracked by the context. This includes entities that you load/query from the database, and entities that were previously added and saved to the database.

Simply modify the values assigned to properties and then call *SaveChanges*.

<!-- [!code-csharp[Main](samples/core/Saving/Saving/Basics/Sample.cs)] -->
````csharp
        using (var db = new BloggingContext())
        {
            var blog = db.Blogs.First();
            blog.Url = "http://sample.com/blog";
            db.SaveChanges();
        }
````

## Deleting Data

使用* DbSet.Remove*方法将删除你的实体类的实例。

Use the *DbSet.Remove* method to delete instances of you entity classes.

如果该实体早已存在数据库中，它将在调用*SaveChanges*时被删除。如果实体尚未保存到数据库（即它是作为跟踪添加）然后将它从上下文中移除，当调用*SaveChanges*时，它将不再被插入。

If the entity already exists in the database, it will be deleted during *SaveChanges*. If the entity has not yet been saved to the database (i.e. it is tracked as added) then it will be removed from the context and will no longer be inserted when *SaveChanges* is called.

<!-- [!code-csharp[Main](samples/core/Saving/Saving/Basics/Sample.cs)] -->
````csharp
        using (var db = new BloggingContext())
        {
            var blog = db.Blogs.First();
            db.Blogs.Remove(blog);
            db.SaveChanges();
        }
````

## Multiple Operations in a single SaveChanges

你可以一个将多个操作 Add/Update/Remove合并在一个*SaveChanges*中。

You can combine multiple Add/Update/Remove operations into a single call to *SaveChanges*.

> [!NOTE]
> 对于大多数数据库提供商来说，*SaveChanges*是一个事务。这意味着所有的操作要么都成功，要么都失败，永远不会只有部分可以成功。

> For most database providers, *SaveChanges* is transactional. This means  all the operations will either succeed or fail and the operations will never be left partially applied.

<!-- [!code-csharp[Main](samples/core/Saving/Saving/Basics/Sample.cs)] -->
````csharp
        using (var db = new BloggingContext())
        {
            db.Blogs.Add(new Blog { Url = "http://sample.com/blog_one" });
            db.Blogs.Add(new Blog { Url = "http://sample.com/blog_two" });

            var firstBlog = db.Blogs.First();
            firstBlog.Url = "";

            var lastBlog = db.Blogs.Last();
            db.Blogs.Remove(lastBlog);

            db.SaveChanges();
        }
````
