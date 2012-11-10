SharpDevelop 4.3 now has support for Code First Migrations using Entity Framework 5.0.

Entity Framework first included [Code First Migrations](http://msdn.microsoft.com/en-us/data/jj591621) in version 4.3 back in February 2012. This allowed database changes to be implemented through code. The Entity Framework NuGet package included three new PowerShell cmdlets that extend the NuGet Package Manager Console and you could use from within Visual Studio.

1. Enable-Migrations: Adds support for migrations to your project.
2. Add-Migration: Generates code for a database migration based on changes to your database model.
3. Update-Database: Applies migrations to the database.

The official EntityFramework NuGet packages are Visual Studio specific, but using the SharpDevelop versions you you can now use these PowerShell cmdlets in SharpDevelop. This has been made possible by [Entity Framework](http://msdn.microsoft.com/en-US/data/ef) being [open sourced](http://entityframework.codeplex.com/).

The EntityFramework.SharpDevelop NuGet package includes the original EntityFramework.dll but has modified versions of the  apart from the  has its own versions of the assemblies containing the PowerShell cmdlets -  EntityFramework.PowerShell.dll and EntityFramework.PowerShell.Utility.dll - which work with SharpDevelop.

Now let us take a look at how to use the EntityFramework.SharpDevelop NuGet package.

##Installation

Before you begin you should have [SQL Express](http://www.microsoft.com/en-us/download/details.aspx?id=1695) installed.

We will look at using Entity Framework with a simple C# console application that adds a blog post to a SQL Express database and reads all the existing blog posts from the database. So in SharpDevelop create a new Console Application. Now open the NuGet Manage Packages dialog by right clicking the project and selecting Manage Packages. Search for the EntityFramework.SharpDevelop package and click the Add button to add it to your project. A reference to the EntityFramework assembly will be added to your project. 

Now we are ready to use Entity Framework. Before we look at database migrations let us write some code to use Entity Framework and access a SQL Express database.

##Using Entity Framework

First we add a new Post class to your project.

	public class Post
	{
		public int Id { get; set; }
		public string Title { get; set; }
		public string Text { get; set; }
	}

Now we add a new database context class.

	public class BloggingContext : DbContext
	{
		public DbSet<Post> Posts { get; set; }
	}

You will also need to add a using statement to the BloggingContext class so the EntityFramework's DbContext class can be found.

        using System.Data.Entity;

Now in your Program.cs add a using statement for System.Linq.

        using System.Linq;

Then in Program.cs update the Main method as shown below:

	class Program
	
	{
		public static void Main(string[] args)
		{
			Console.Write("Enter a post title: ");
			string title = Console.ReadLine();
			
			Console.Write("Enter the post text: ");
			string text = Console.ReadLine();
			
			using (var db = new BloggingContext()) {
				
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

Compile and run this application and you should see it add a new post to the database and then read it back again.

		Enter a post title: First Post
		Enter the post text: Some text here...
		All posts in the database:
		Post.Title: First Post
		Press a key to exit.

After running this Entity Framework will have created a new database with the same name as the db context class you created. In the screenshot below you can see the database open in [SQL Server Express Management Studio](http://www.microsoft.com/en-us/download/details.aspx?id=22985).

![Code first database created by Entity Framework](/Images/CodeFirstDatabaseCreatedByEntityFramework.png)

You can see that Entity Framework has created a Posts table as well as a __MigrationHistory table.

##Enabling Migrations

Now we will enable database migrations. Open the Package Management Console and enter the command.

        Enable-Migrations

This adds a Migrations folder to your project along with a Configuration.cs file and an initial migration.

![Project items after migrations are enabled](/Images/ProjectItemsAfterCodeFirstMigrationsEnabled.png)

The Configuration.cs file can be edited to add seed data and alter other settings used when migrating the database.

The initial migration files contain code to generate the Posts table in your database.

Now let us take a look at adding a new migration.

##Adding a Migration

Now we want to add a Published Date to our Post class. Add the new PublishedDate property to the Post class as shown below.

	public class Post
	{
		public int Id { get; set; }
		public string Title { get; set; }
		public string Text { get; set; }
		public DateTime PublishedDate { get; set; }
	}

Now we can generate a code first migration by entering the following command into the Package Management Console.

        Add-Migration AddPublishedDate

This creates a new migration and gives it the name AddPublishedDate. A new file will be added to your migrations folder which contains the code first migration for the new property.

	namespace EFCodeFirst.Migrations
	{
	    using System;
	    using System.Data.Entity.Migrations;
	    
	    public partial class AddPublishedDate : DbMigration
	    {
	        public override void Up()
	        {
	            AddColumn("dbo.Posts", "PublishedDate", c => c.DateTime(nullable: false));
	        }
	        
	        public override void Down()
	        {
	            DropColumn("dbo.Posts", "PublishedDate");
	        }
	    }
	}

Now we can update our database.

##Updating the Database

To update the database we run the following command in the Package Management Console window.

        Update-Database

The new PublishedDate column will then be added to the Posts database table.

![Posts table after code first migration applied](/Images/PostsTableAfterCodeFirstMigrationApplied.png)

That concludes the introduction to using Code First Migrations with SharpDevelop.

## Changes Made to Original Entity Framework NuGet Packages

1. Renamed NuGet package to EntityFramework.SharpDevelop and included files and assemblies from official EF NuGet package.
2. Recompiled the EntityFramework.PowerShell and EntityFramework.PowerShell.Utility projects against SharpDevelop's EnvDTE implementation.
3. Small modifications to the project to compile against .NET 4.0 and the EntityFramework.dll. Microsoft's released source code on Codeplex is for EntityFramework 6 and not for EF 5.
4. EntityFramework.PowerShell and EntityFramework.PowerShell.Utility assemblies are no longer signed.
5. Added workaround to fix AppDomain.CreateInstance failing to find constructors taking a EnvDTE.Project parameter.
6. Add SharpDevelop.EnvDTE.dll to NuGet packages.

All code changes are available on [GitHub](https://github.com/mrward/entityframework-sharpdevelop).
