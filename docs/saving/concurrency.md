---
title: Concurrency Conflicts | Microsoft Docs
author: rowanmiller
ms.author: divega

ms.date: 10/27/2016

ms.assetid: bce0539d-b0cd-457d-be71-f7ca16f3baea
ms.technology: entity-framework-core
 
uid: core/saving/concurrency
---
# Concurrency Conflicts

如果将属性配置为并发令牌，则EF将检查在保存该记录的更改时，没有其他用户修改该数据库中的值。

If a property is configured as a concurrency token then EF will check that no other user has modified that value in the database when saving changes to that record.

> [!TIP]
> 你可以在GitHub查看这篇文章的[示例代码](https://github.com/aspnet/EntityFramework.Docs/tree/master/samples/core/Saving/Saving/Concurrency/)。

> You can view this article's [sample](https://github.com/aspnet/EntityFramework.Docs/tree/master/samples/core/Saving/Saving/Concurrency/) on GitHub.

## How concurrency works in EF

对EF Core并发工作详情请移步[Concurrency Tokens](../modeling/concurrency.html)。

For a detailed description of how concurrency works in Entity Framework Core, see [Concurrency Tokens](../modeling/concurrency.md).

## Resolving concurrency conflicts

解决并发冲突涉及使用一个算法将当前用户的挂起更改与数据库中的更改合并。根据您的应用程序的确切方法会有所不同，但一个常见的方法是向用户显示值，并让他们决定正确的值存储在数据库中。

Resolving a concurrency conflict involves using an algorithm to merge the pending changes from the current user with the changes made in the database. The exact approach will vary based on your application, but a common approach is to display the values to the user and have them decide the correct values to be stored in the database.

**有三种可用的值来帮助解决并发冲突。**

**There are three sets of values available to help resolve a concurrency conflict.**


* **当前值**是应用程序试图写入数据库的值。

* **Current values** are the values that the application was attempting to write to the database.

* **原始值**是在任何编辑之前从数据库中检索的值。

* **Original values** are the values that were originally retrieved from the database, before any edits were made.

***数据库值**是当前存储在数据库中的值。

* **Database values** are the values currently stored in the database.

处理并发冲突，在`SaveChanges()`捕获`DbUpdateConcurrencyException`异常，利用`DbUpdateConcurrencyException.Entries`重新设置受影响的实体，然后重试`SaveChanges()`操作。

To handle a concurrency conflict, catch a `DbUpdateConcurrencyException` during `SaveChanges()`, use `DbUpdateConcurrencyException.Entries` to prepare a new set of changes for the affected entities, and then retry the `SaveChanges()` operation.

在下面的示例中，`Person.FirstName`和`Person.LastName`被设置为并发标记。在`// TODO:`注释处，你可以将包含特定于应用程序的逻辑来选择要保存到数据库中的值。

In the following example, `Person.FirstName` and `Person.LastName` are setup as concurrency token. There is a `// TODO:` comment in the location where you would include application specific logic to choose the value to be saved to the database.

<!-- [!code-csharp[Main](samples/core/Saving/Saving/Concurrency/Sample.cs?highlight=53,54)] -->
````csharp
using Microsoft.EntityFrameworkCore;
using System;
using System.ComponentModel.DataAnnotations;
using System.Linq;

namespace EFSaving.Concurrency
{
    public class Sample
    {
        public static void Run()
        {
            // Ensure database is created and has a person in it
            using (var context = new PersonContext())
            {
                context.Database.EnsureDeleted();
                context.Database.EnsureCreated();

                context.People.Add(new Person { FirstName = "John", LastName = "Doe" });
                context.SaveChanges();
            }

            using (var context = new PersonContext())
            {
                // Fetch a person from database and change phone number
                var person = context.People.Single(p => p.PersonId == 1);
                person.PhoneNumber = "555-555-5555";

                // Change the persons name in the database (will cause a concurrency conflict)
                context.Database.ExecuteSqlCommand("UPDATE dbo.People SET FirstName = 'Jane' WHERE PersonId = 1");

                try
                {
                    // Attempt to save changes to the database
                    context.SaveChanges();
                }
                catch (DbUpdateConcurrencyException ex)
                {
                    foreach (var entry in ex.Entries)
                    {
                        if (entry.Entity is Person)
                        {
                            // Using a NoTracking query means we get the entity but it is not tracked by the context
                            // and will not be merged with existing entities in the context.
                            var databaseEntity = context.People.AsNoTracking().Single(p => p.PersonId == ((Person)entry.Entity).PersonId);
                            var databaseEntry = context.Entry(databaseEntity);

                            foreach (var property in entry.Metadata.GetProperties())
                            {
                                var proposedValue = entry.Property(property.Name).CurrentValue;
                                var originalValue = entry.Property(property.Name).OriginalValue;
                                var databaseValue = databaseEntry.Property(property.Name).CurrentValue;

                                // TODO: Logic to decide which value should be written to database
                                // entry.Property(property.Name).CurrentValue = <value to be saved>;

                                // Update original values to
                                entry.Property(property.Name).OriginalValue = databaseEntry.Property(property.Name).CurrentValue;
                            }
                        }
                        else
                        {
                            throw new NotSupportedException("Don't know how to handle concurrency conflicts for " + entry.Metadata.Name);
                        }
                    }

                    // Retry the save operation
                    context.SaveChanges();
                }
            }
        }

        public class PersonContext : DbContext
        {
            public DbSet<Person> People { get; set; }

            protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
            {
                optionsBuilder.UseSqlServer(@"Server=(localdb)\mssqllocaldb;Database=EFSaving.Concurrency;Trusted_Connection=True;");
            }
        }

        public class Person
        {
            public int PersonId { get; set; }

            [ConcurrencyCheck]
            public string FirstName { get; set; }

            [ConcurrencyCheck]
            public string LastName { get; set; }

            public string PhoneNumber { get; set; }
        }

    }
}
````
