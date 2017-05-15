---
title: Related Data | Microsoft Docs
author: rowanmiller
ms.author: divega

ms.date: 10/27/2016

ms.assetid: 07b6680f-ffcf-412c-9857-f997486b386c
ms.technology: entity-framework-core
 
uid: core/saving/related-data
---
# Related Data

除了隔离实体之外，你还可以在模型使用关系定义。
In addition to isolated entities, you can also make use of the relationships defined in your model.

> [!TIP]
> 你可以在GitHub查看这篇的[示例代码](https://github.com/aspnet/EntityFramework.Docs/tree/master/samples/core/Saving/Saving/RelatedData/)。
> You can view this article's [sample](https://github.com/aspnet/EntityFramework.Docs/tree/master/samples/core/Saving/Saving/RelatedData/) on GitHub.

## Adding a graph of new entities

如果创建了几个新的相关实体，将其中的一个添加到上下文将会导致其他的相关实体也被添加。
If you create several new related entities, adding one of them to the context will cause the others to be added too.

在下面的示例中，blog和三个相关posts将全部会被插入到数据库中。该posts被发现和添加，是因为EF Core可以通过`Blog.Posts`的导航属性。

In the following example, the blog and three related posts are all inserted into the database. The posts are found and added, because they are reachable via the `Blog.Posts` navigation property.

<!-- [!code-csharp[Main](samples/core/Saving/Saving/RelatedData/Sample.cs)] -->
````csharp
        using (var context = new BloggingContext())
        {
            var blog = new Blog
            {
                Url = "http://blogs.msdn.com/dotnet",
                Posts = new List<Post>
                {
                    new Post { Title = "Intro to C#" },
                    new Post { Title = "Intro to VB.NET" },
                    new Post { Title = "Intro to F#" }
                }
            };

            context.Blogs.Add(blog);
            context.SaveChanges();
        }
````

## Adding a related entity

如果从上下文已经跟踪的实体的导航属性中引用一个新实体，则该实体将被发现并插入数据库中。

If you reference a new entity from the navigation property of an entity that is already tracked by the context, the entity will be discovered and inserted into the database.

在下面的示例中，`post`实体被插入是因为它被添加到从数据库中获取的`blog`实体的`Posts属性中。

In the following example, the `post` entity is inserted because it is added to the `Posts` property of the `blog` entity which was fetched from the database.

<!-- [!code-csharp[Main](samples/core/Saving/Saving/RelatedData/Sample.cs)] -->
````csharp
        using (var context = new BloggingContext())
        {
            var blog = context.Blogs.First();
            var post = new Post { Title = "Intro to EF Core" };

            blog.Posts.Add(post);
            context.SaveChanges();
        }
````

## Changing relationships

如果更改实体的导航属性，则将对数据库中的外键列进行相应的更改。

If you change the navigation property of an entity, the corresponding changes will be made to the foreign key column in the database.

在下面的例子中，`post`实体将会被更新，属于新的`blog`实体，因为它的`Blog`导航属性设置为指向`blog`。请注意，`blog`也将被插入到数据库中，因为它是一个新实体，由一个实体的导航属性所引用，该实体已经被上下文跟踪(`post`)。

In the following example, the `post` entity is updated to belong to the new `blog` entity because its `Blog` navigation property is set to point to `blog`. Note that `blog` will also be inserted into the database because it is a new entity that is referenced by the navigation property of an entity that is already tracked by the context (`post`).

<!-- [!code-csharp[Main](samples/core/Saving/Saving/RelatedData/Sample.cs)] -->
````csharp
        using (var context = new BloggingContext())
        {
            var blog = new Blog { Url = "http://blogs.msdn.com/visualstudio" };
            var post = context.Posts.First();

            blog.Posts.Add(post);
            context.SaveChanges();
        }
````

## Removing relationships

可以通过将引用导航设置为`null`或从集合导航中移除相关实体来移除关联。

You can remove a relationship by setting a reference navigation to `null`, or removing the related entity from a collection navigation.

如果通过级联删除配置，子/依赖实体将从数据库中删除，请查看[级联删除](cascade-delete.html)了解更多的信息。如果不是通过配置设置级联删除，数据库中的外键列将被设置为null（如果列不接受null，将抛出一个异常）。

If a cascade delete is configured, the child/dependent entity will be deleted from the database, see [Cascade Delete](cascade-delete.html) for more information. If no cascade delete is configured, the foreign key column in the database will be set to null (if the column does not accept nulls, an exception will be thrown).

在下面的例子中，一个级联删除配置在`Blog`和`Post`之间的关联，所以`post`实体将从数据库中被删除。

In the following example, a cascade delete is configured on the relationship between `Blog` and `Post`, so the `post` entity is deleted from the database.

<!-- [!code-csharp[Main](samples/core/Saving/Saving/RelatedData/Sample.cs)] -->
````csharp
        using (var context = new BloggingContext())
        {
            var blog = context.Blogs.Include(b => b.Posts).First();
            var post = blog.Posts.First();

            blog.Posts.Remove(post);
            context.SaveChanges();
        }
````
