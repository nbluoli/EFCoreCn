---
title: Transactions | Microsoft Docs
author: rowanmiller
ms.author: divega

ms.date: 10/27/2016

ms.assetid: d3e6515b-8181-482c-a790-c4a6778748c1
ms.technology: entity-framework-core
 
uid: core/saving/transactions
---
# Transactions

事务允许以原子方式处理多个数据库操作。如果事务已提交，则所有操作都成功地应用于数据库。如果事务回滚，则操作将不会应用于数据库。

Transactions allow several database operations to be processed in an atomic manner. If the transaction is committed, all of the operations are successfully applied to the database. If the transaction is rolled back, none of the operations are applied to the database.

> [!TIP]
你可以在GitHub上查看本文的[示例代码](https://github.com/aspnet/EntityFramework.Docs/tree/master/samples/core/Saving/Saving/Transactions/)。

> You can view this article's [sample](https://github.com/aspnet/EntityFramework.Docs/tree/master/samples/core/Saving/Saving/Transactions/) on GitHub.

## Default transaction behavior

默认情况下，如果数据库提供程序支持事务，在单独调用`SaveChanges()`时，所有改变都将应用在一个事务中。如果任何更改失败，则回滚事务，并且不会将更改应用于数据库。这意味着`SaveChanges()`保证要么完全成功，或者如果发生错误，要么完全不更改数据库的离开。

By default, if the database provider supports transactions, all changes in a single call to `SaveChanges()` are applied in a transaction. If any of the changes fail, then the transaction is rolled back and none of the changes are applied to the database. This means that `SaveChanges()` is guaranteed to either completely succeed, or leave the database unmodified if an error occurs.

对于大多数应用程序，默认行为是足够的。如果应用程序强制认为需要，则只能手动控制事务。

For most applications, this default behavior is sufficient. You should only manually control transactions if your application requirements deem it necessary.

## Controlling transactions

你可以使用`DbContext.Database` API开始、提交和回滚事务。下面的示例显示两个`SaveChanges()`操作和一个Linq 查询单个事务执行中被执行。

You can use the `DbContext.Database` API to begin, commit, and rollback transactions. The following example shows two `SaveChanges()` operations and a LINQ query being executed in a single transaction.

并非所有数据库提供程序都支持事务。当事务API被调用时，一些提供者可能会抛出异常或no-op。

Not all database providers support transactions. Some providers may throw or no-op when transaction APIs are called.

<!-- [!code-csharp[Main](samples/core/Saving/Saving/Transactions/ControllingTransaction/Sample.cs?highlight=3,17,18,19)] -->
````csharp
        using (var context = new BloggingContext())
        {
            using (var transaction = context.Database.BeginTransaction())
            {
                try
                {
                    context.Blogs.Add(new Blog { Url = "http://blogs.msdn.com/dotnet" });
                    context.SaveChanges();

                    context.Blogs.Add(new Blog { Url = "http://blogs.msdn.com/visualstudio" });
                    context.SaveChanges();

                    var blogs = context.Blogs
                        .OrderBy(b => b.Url)
                        .ToList();

                    // Commit transaction if all commands succeed, transaction will auto-rollback
                    // when disposed if either commands fails
                    transaction.Commit();
                }
                catch (Exception)
                {
                    // TODO: Handle failure
                }
            }
        }
````

## Cross-context transaction (relational databases only)

您还可以在多个上下文实例共享事务。此功能仅可使用关系数据库提供商，因为它需要调用`DbTransaction`和`DbConnection`类，这是针对关系数据库。

You can also share a transaction across multiple context instances. This functionality is only available when using a relational database provider because it requires the use of `DbTransaction` and `DbConnection`, which are specific to relational databases.

为共享一个事务，上下文一定也要共享同一个`DbConnection`和同一个`DbTransaction`。

To share a transaction, the contexts must share both a `DbConnection` and a `DbTransaction`.

### Allow connection to be externally provided

在一个上下文中构建一个共享的`DbConnection`，需确保有足够的连接。

Sharing a `DbConnection` requires the ability to pass a connection into a context when constructing it.

最简单的办法就是允许外部提供`DbConnection`，在上下文配置中停止使用`DbContext.OnConfiguring`方法，通过外部创建` DbContextOptions`然后传给上下文构造函数。

The easiest way to allow `DbConnection` to be externally provided, is to stop using the `DbContext.OnConfiguring` method to configure the context and externally create `DbContextOptions` and pass them to the context constructor.

> [!TIP]
> 以前你使用`DbContext.OnConfiguring`API的`DbContextOptionsBuilder`去配置上下文，你现在可以使用外部创建的`DbContextOptions`。

> `DbContextOptionsBuilder` is the API you used in `DbContext.OnConfiguring` to configure the context, you are now going to use it externally to create `DbContextOptions`.

<!-- [!code-csharp[Main](samples/core/Saving/Saving/Transactions/SharingTransaction/Sample.cs?highlight=3,4,5)] -->
````csharp
    public class BloggingContext : DbContext
    {
        public BloggingContext(DbContextOptions<BloggingContext> options)
            : base(options)
        { }

        public DbSet<Blog> Blogs { get; set; }
    }
````
或者继续使用`DbContext.OnConfiguring`，但使用`DbContext.OnConfiguring`需接受一个`DbConnection`，并保存。

An alternative is to keep using `DbContext.OnConfiguring`, but accept a `DbConnection` that is saved and then used in `DbContext.OnConfiguring`.

<!-- literal_block"ids  "classes  "xml:space": "preserve", "backrefs  "linenos": false, "dupnames  : "csharp", highlight_args}, "names": [] -->
````csharp
public class BloggingContext : DbContext
{
    private DbConnection _connection;

    public BloggingContext(DbConnection connection)
    {
      _connection = connection;
    }

    public DbSet<Blog> Blogs { get; set; }

    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        optionsBuilder.UseSqlServer(_connection);
    }
}
````

### Share connection and transaction

现在可以创建共享同一连接的多个上下文实例。然后用`DbContext.Database.UseTransaction(DbTransaction)` API实现两个上下文在同一个事务中。

You can now create multiple context instances that share the same connection. Then use the `DbContext.Database.UseTransaction(DbTransaction)` API to enlist both contexts in the same transaction.

<!-- [!code-csharp[Main](samples/core/Saving/Saving/Transactions/SharingTransaction/Sample.cs?highlight=1,2,3,7,16,23,24,25)] -->
````csharp
        var options = new DbContextOptionsBuilder<BloggingContext>()
            .UseSqlServer(new SqlConnection(connectionString))
            .Options;

        using (var context1 = new BloggingContext(options))
        {
            using (var transaction = context1.Database.BeginTransaction())
            {
                try
                {
                    context1.Blogs.Add(new Blog { Url = "http://blogs.msdn.com/dotnet" });
                    context1.SaveChanges();

                    using (var context2 = new BloggingContext(options))
                    {
                        context2.Database.UseTransaction(transaction.GetDbTransaction());

                        var blogs = context2.Blogs
                            .OrderBy(b => b.Url)
                            .ToList();
                    }

                    // Commit transaction if all commands succeed, transaction will auto-rollback
                    // when disposed if either commands fails
                    transaction.Commit();
                }
                catch (Exception)
                {
                    // TODO: Handle failure
                }
            }
        }
````

## Using external DbTransactions (relational databases only)

如果使用多个数据访问技术访问关系数据库，则可能需要在这些不同技术之间执行的事务之间共享事务。

If you are using multiple data access technologies to access a relational database, you may want to share a transaction between operations performed by these different technologies.

下面的示例，演示如何执行一个ADO.NET SqlClient操作和Entity Framework Core操作在同一事务中。

The following example, shows how to perform an ADO.NET SqlClient operation and an Entity Framework Core operation in the same transaction.

<!-- [!code-csharp[Main](samples/core/Saving/Saving/Transactions/ExternalDbTransaction/Sample.cs?highlight=4,10,21,26,27,28)] -->
````csharp
        var connection = new SqlConnection(connectionString);
        connection.Open();

        using (var transaction = connection.BeginTransaction())
        {
            try
            {
                // Run raw ADO.NET command in the transaction
                var command = connection.CreateCommand();
                command.Transaction = transaction;
                command.CommandText = "DELETE FROM dbo.Blogs";
                command.ExecuteNonQuery();

                // Run an EF Core command in the transaction
                var options = new DbContextOptionsBuilder<BloggingContext>()
                    .UseSqlServer(connection)
                    .Options;

                using (var context = new BloggingContext(options))
                {
                    context.Database.UseTransaction(transaction);
                    context.Blogs.Add(new Blog { Url = "http://blogs.msdn.com/dotnet" });
                    context.SaveChanges();
                }

                // Commit transaction if all commands succeed, transaction will auto-rollback
                // when disposed if either commands fails
                transaction.Commit();
            }
            catch (System.Exception)
            {
                // TODO: Handle failure
            }
````
