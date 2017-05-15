---
title: Cascade Delete | Microsoft Docs
author: rowanmiller
ms.author: divega

ms.date: 10/27/2016

ms.assetid: ee8e14ec-2158-4c9c-96b5-118715e2ed9e
ms.technology: entity-framework-core
 
uid: core/saving/cascade-delete
---
# Cascade Delete

级联删除允许删除主/父实体对依赖/子实体相关的副作用。

Cascade delete allows deletion of a principal/parent entity to have a side effect on dependent/child entities it is related to.

**以下是三个级联删除行为:**

**There are three cascade delete behaviors:**

* **级联:** 依赖实体也被删除。

* **Cascade:** Dependent entities are also deleted.

* **设置空:** 依赖实体的外键属性设置为`null`。

* **SetNull:** The foreign key properties in dependent entities are set to null.

* **约束:** 删除操作不应用于依赖实体，依赖保持不变。

* **Restrict:** The delete operation is not applied to dependent entities. The dependent entities remain unchanged.

请查看[Relationships](../modeling/relationships.html)了解更多关于级联删除的约定和配置信息。

See [Relationships](../modeling/relationships.html) for more information about conventions and configuration for cascade delete.

> [!TIP]
> 你也可以在GitHub查看这篇文章的[示例代码](https://github.com/aspnet/EntityFramework.Docs/tree/master/samples/core/Saving/Saving/CascadeDelete/)。

> You can view this article's [sample](https://github.com/aspnet/EntityFramework.Docs/tree/master/samples/core/Saving/Saving/CascadeDelete/) on GitHub.

## Cascading to tracked entities

当你调用*SaveChanges*时，级联删除规则将会被应用于任何被上下文跟踪的实体。

When you call *SaveChanges*, the cascade delete rules will be applied to any entities that are being tracked by the context.

考虑一个简单的和*Blog*和*Post*模型，这两个实体必然是关联。按照约定。这种关系的级联行为被设置为*Cascade*。

Consider a simple *Blog* and *Post* model where the relationship between the two entities is required. By convention. the cascade behavior for this relationship is set to *Cascade*.

下面的代码从数据库中加载Blog及其所有相关的Posts（使用*Include*方法）。然后删除Blog。

The following code loads a Blog and all its related Posts from the database (using the *Include* method). The code then deletes the Blog.

<!-- [!code-csharp[Main](samples/core/Saving/Saving/CascadeDelete/Sample.cs)] -->
````csharp
        using (var db = new BloggingContext())
        {
            var blog = db.Blogs.Include(b => b.Posts).First();
            db.Remove(blog);
            db.SaveChanges();
        }
````
因为所有的Posts都是由上下文跟踪的，所以在保存到数据库之前，将级联行为应用到它们上。EF因此为每个实体发出*DELETE*声明。

Because all the Posts are tracked by the context, the cascade behavior is applied to them before saving to the database. EF therefore issues a  *DELETE* statement for each entity.

<!-- literal_block"xml:space": "preserve", "classes  "backrefs  "names  "dupnames   -->
````
   DELETE FROM [Post]
   WHERE [PostId] = @p0;
   DELETE FROM [Post]
   WHERE [PostId] = @p1;
   DELETE FROM [Blog]
   WHERE [BlogId] = @p2;
````

## Cascading to untracked entities

下面的代码与我们以前的示例几乎相同，但它不从数据库加载相关的Posts。

The following code is almost the same as our previous example, except it does not load the related Posts from the database.

<!-- [!code-csharp[Main](samples/core/Saving/Saving/CascadeDelete/Sample.cs)] -->
````csharp
        using (var db = new BloggingContext())
        {
            var blog = db.Blogs.First();
            db.Remove(blog);
            db.SaveChanges();
        }
````

由于Posts没有被上下文跟踪，所以*DELETE*声明只是针对*Blog*。这依赖于数据库中存在的相应的级联行为，以确保不被上下文跟踪的数据也被删除。如果您使用EF创建数据库，此级联行为EF将为您设置。

Because the Posts are not tracked by the context, a *DELETE* statement is only issued for the *Blog*. This relies on a corresponding cascade behavior being present in the database to ensure data that is not tracked by the context is also deleted. If you use EF to create the database, this cascade behavior will be setup for you.

<!-- literal_block"xml:space": "preserve", "classes  "backrefs  "names  "dupnames   -->
````
   DELETE FROM [Blog]
   WHERE [BlogId] = @p0;
````
