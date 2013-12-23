SharpDevelop 4.4 now has support for Code First Migrations using Entity Framework 6.0.2.

On NuGet there is an [EntityFramework.SharpDevelop NuGet package](https://www.nuget.org/packages/EntityFramework.SharpDevelop) that includes the original Entity Framework assemblies along with custom versions of the PowerShell cmdlets, listed below, that work with SharpDevelop

1. Enable-Migrations: Adds support for migrations to your project.
2. Add-Migration: Generates code for a database migration based on changes to your database model.
3. Update-Database: Applies migrations to the database.
4. Get-Migrations: Shows the migrations that have been applied to the database.

Now let us take a look at how to use the EntityFramework.SharpDevelop NuGet package.

##Installation

Before you begin you should have [SQL Express](http://www.microsoft.com/en-us/download/details.aspx?id=1695) installed. You will also need [SharpDevelop version 4.4.0.9722](http://build.sharpdevelop.net/buildartefacts/) or above.

We will look at using Entity Framework in a simple C# console application. This application will add a blog post to a SQL Express database and then read all the existing blog posts in the database. We used this example before in a [previous blog post](http://community.sharpdevelop.net/blogs/mattward/archive/2012/11/11/CodeFirstMigrationsWithEntityFramework.aspx) when using Entity Framework with SharpDevelop so we will not go into as much detail as before, instead concentrating on some of the new features that Entity Framework brings, such as logging and support for stored procedures.

In SharpDevelop first create a new C# Console Application. Add the EntityFramework.SharpDevelop NuGet package to the project either by using the Manage Packages dialog or by using the Package Management Console window.

##Using Entity Framework

Here is the code we will start with.

	using System;
	using System.Data.Entity;
	using System.Linq;
	
	namespace EFCodeFirst
	{
		public class Post
		{
			public int Id { get; set; }
			public string Title { get; set; }
			public string Text { get; set; }
		}
	
		public class BloggingContext : DbContext
		{
			public DbSet<Post> Posts { get; set; }
		}
	
		class Program
		{
			public static void Main(string[] args)
			{
				Console.Write("Enter a post title: ");
				string title = Console.ReadLine();
				
				Console.Write("Enter the post text: ");
				string text = Console.ReadLine();
				
				using (var db = new BloggingContext()) {
					
					db.Database.Log = Console.Write;
					
					var newPost = new Post {
						Title = title,
						Text = text
					};
					
					db.Posts.Add(newPost);
					db.SaveChanges();
					
					Console.WriteLine("All posts in the database:");
	
					IQueryable<Post> query = db.Posts
						.OrderBy(p => p.Title)
						.Select(p => p);
					
					foreach (Post post in query) {
						Console.WriteLine("Post.Title: " + post.Title);
					}
				}
				
				Console.WriteLine("Press a key to exit.");
				Console.ReadKey();
			}
		}
	}

The above code uses the new logging feature of Entity Framework 6. The line after the database context is created configures Entity Framework to write information, such as the queries executed, to the console:

    db.Database.Log = Console.Write;

Running the console application will cause Entity Framework to create a new database, add a new post to the database, and then read all the posts from the database. You will also see information logged by Entity Framework:

    INSERT [dbo].[Posts]([Title], [Text])
    VALUES (@0, @1)
    SELECT [Id]
    FROM [dbo].[Posts]
    WHERE @@ROWCOUNT > 0 AND [Id] = scope_identity()
    -- @0: 'first' (Type = String, Size = -1)
    -- @1: 'first post' (Type = String, Size = -1)
    -- Executing at 21/12/2013 12:09:06 +00:00
    -- Completed in 11 ms with result: SqlDataReader
    
    All posts in the database:
    SELECT
        [Extent1].[Id] AS [Id],
        [Extent1].[Title] AS [Title],
        [Extent1].[Text] AS [Text]
        FROM [dbo].[Posts] AS [Extent1]
        ORDER BY [Extent1].[Title] ASC
    -- Executing at 23/12/2013 12:09:06 +00:00
    -- Completed in 2 ms with result: SqlDataReader

##Database Migrations

Now let us take a look at adding a new migration. Open the Package Management Console and run the following command to configure database migrations for the project.

        Enable-Migrations

 We will now use the new stored procedure support that was added to Entity Framework 6. Instead of inserting, updating and deleting posts in the database using SQL we will use stored procedures. To do this we modify the database context as shown below.

	public class BloggingContext : DbContext
	{
		public DbSet<Post> Posts { get; set; }
		
		protected override void OnModelCreating(DbModelBuilder modelBuilder)
		{
			modelBuilder
				.Entity<Post>()
				.MapToStoredProcedures();
		}
	}

Here we have overridden the OnModelCreating method and configured Entity Framework to use stored procedures for our Posts.

Now generate a code first migration for the stored procedures by running the following command in the Package Management Console.

        Add-Migration StoredProcs

This creates a new migration that includes a set of simple stored procedures for inserting, updating and deleting Posts in our database. Part of this generated code is shown below:

    public partial class StoredProcs : DbMigration
    {
        public override void Up()
        {
            CreateStoredProcedure(
                "dbo.Post_Insert",
                p => new
                    {
                        Title = p.String(),
                        Text = p.String(),
                    },
                body:
                    @"INSERT [dbo].[Posts]([Title], [Text])
                      VALUES (@Title, @Text)
                      
                      DECLARE @Id int
                      SELECT @Id = [Id]
                      FROM [dbo].[Posts]
                      WHERE @@ROWCOUNT > 0 AND [Id] = scope_identity()
                      
                      SELECT t0.[Id]
                      FROM [dbo].[Posts] AS t0
                      WHERE @@ROWCOUNT > 0 AND t0.[Id] = @Id"
            );

Now you can add the new stored procedures to your database by running the following command in the Package Management Console window.

        Update-Database

The stored procedures can be seen using SQL Management Studio.

![Stored procedures created by Entity Framework](/Images/EntityFramework6GeneratedStoredProcedures.png)

If you run the console application now you will see the stored procedures being used when a new post is created in the console output.

    [dbo].[Post_Insert]
    -- Title: 'second' (Type = String, Size = 1073741823)
    -- Text: 'second post' (Type = String, Size = 1073741823)
    -- Executing at 23/12/2013 12:21:47 +00:00
    -- Completed in 5 ms with result: SqlDataReader

That concludes our look at using Entity Framework 6 code first migrations with SharpDevelop. 

## Further Information on Entity Framework

 1. [Entity Framework on the MSDN](http://msdn.microsoft.com/data/ef). 
 2. [Entity Framework 6: Database Access Anywhere, Easily](http://channel9.msdn.com/Events/TechEd/NorthAmerica/2013/DEV-B335) - Rowan Miller's Tech Ed 2013 Talk.

The code for the modified NuGet packages is available on [GitHub](https://github.com/mrward/entityframework-sharpdevelop/tree/ef6.0.2-nuget).
