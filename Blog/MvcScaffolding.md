SharpDevelop 4.3 now has support for [MVC Scaffolding](http://mvcscaffolding.codeplex.com).

[MVC Scaffolding](http://mvcscaffolding.codeplex.com/), maintained by [Steve Sanderson](http://blog.stevensanderson.com), provides a way to quickly generate code for views and controllers in your ASP.NET MVC application. It has support for scaffolding the following:

* Views and controllers.
* Entity Framework DbContexts.
* Repositories.
* Controller actions.
* Unit tests.

It makes use of T4 templates, which can be customised, and can be extended with custom scaffolders to support more than just the features listed above.

The original MVC Scaffolding NuGet packages made heavy use of the Visual Studio API and have been modified to work with SharpDevelop. Full details of the changes made to the original NuGet packages can be found at the end of this post.

Steve Sanderson has a [great series of posts on MVC Scaffolding](http://blog.stevensanderson.com/2011/01/13/scaffold-your-aspnet-mvc-3-project-with-the-mvcscaffolding-package/) that covers more than this post will. Now let us take a look at how to use MVC Scaffolding with SharpDevelop.

##Installation

MVC Scaffolding for SharpDevelop is available as a [NuGet package](http://nuget.org/packages/MvcScaffolding.SharpDevelop). The NuGet package you should download is **MvcScaffolding.SharpDevelop**. Install the MvcScaffolding.SharpDevelop NuGet package either from the NuGet package management console or by using the Manage Packages dialog.

Note that the NuGet package MvcScaffolding.SharpDevelop.1.0.9 requires SharpDevelop 4.3.0.9134 or above. If you have an older version installed then you should uninstall the MvcScaffolding.SharpDevelop package, before installing the latest version of SharpDevelop, and then reinstall the MvcScaffolding.SharpDevelop package. A breaking change made to SharpDevelop 4.3 to support the EntityFramework package will cause the older MvcScaffolding.SharpDevelop NuGet package to fail to uninstall cleanly in the newer version of SharpDevelop.

##Scaffolding a Controller and Views

As an example we will create a simple blogging site. Create a Razor MVC application called BloggingSite. Then create the following class in a Models folder.

    using System;

    namespace BloggingSite.Models
    {
        public class Post
        {
            public int Id { get; set; }
            public string Title { get; set; }
            public string Text { get; set; }
            public DateTime PublishedDate { get; set; }
        }
    }

Now we can scaffold a controller and a set of Create, Read, Update and Delete (CRUD) views. In the Package Management console run the following command.

    Scaffold Controller Post

This will generate a set of views, a controller and an Entity Framework database context.

![Scaffold Controller Post output in Package Management Console](/Images/PackageManagementConsoleOutputForScaffoldControllerPost.png)

To run the Blogging Site application you will need to have IIS Express and SQL Server Express installed. Your project will also need to be configured to use IIS Express. Select **Project Options** from the **Project** menu and then select the **Web** tab. Choose IIS Express and then click the **Create application/virtual directory** button.

Then run your application and visit the **/Posts** page in your browser.

![Scaffolded Posts page with empty database](/Images/ScaffoldedPostsPageWithEmptyDatabase.png)

You can then create a new post by clicking the **Create New** link.

![Creating new blog post page](/Images/ScaffoldedPostsCreateNewPostPage.png)

After creating a new post then the post will be displayed on the original /Posts page.

![Scaffolded Posts page with one post](/Images/ScaffoldedPostsPageWithOnePost.png)

Also scaffolded is the details page which you can view by clicking the **Details** link.

![Scaffolded Details page](/Images/ScaffoldedPostsDetailsPage.png)

Finally there is the Delete page where you can remove a post.

![Scaffolded delete post page](/Images/ScaffoldedPostsDeletePage.png)

Scaffolding CRUD views is just one part of MvcScaffolding so let us take a look at some of other scaffolding that is supported. Steve Sanderson goes into more detail on [what you can do with each of the MVC Scaffolders](http://blog.stevensanderson.com/2011/01/13/mvcscaffolding-standard-usage/) than will be covered here.

## Scaffolding a Repository

The PostsController generated uses the BloggingSiteContext class which is derived from Entity Framework's DbContext. 

    public class PostsController : Controller
    {
        private BloggingSiteContext context = new BloggingSiteContext();

        //
        // GET: /Posts/

        public ViewResult Index()
        {
            return View(context.Posts.ToList());
        }

To allow unit testing of the controller we can instead using a repository interface which can be mocked. To do that we run the following command.

    Scaffold Controller Post -Repository -Force

By default MVC Scaffolding will not overwrite views and controllers previously created in your project. In order to overwrite the views and controllers created previously we use the **-Force** option.

The PostsController now looks like this.

    public class PostsController : Controller
    {
        private readonly IPostRepository postRepository;

        // If you are using Dependency Injection, you can delete the following constructor
        public PostsController() : this(new PostRepository())
        {
        }

        public PostsController(IPostRepository postRepository)
        {
            this.postRepository = postRepository;
        }

        //
        // GET: /Posts/

        public ViewResult Index()
        {
            return View(postRepository.All);
        }

##Scaffolding an Empty View

You can scaffold an empty view by using the **View** scaffolder and specifying the name of the controller, without the Controller part, followed by the name of the view.

    Scaffold View Posts MyEmptyView

In a Razor MVC application this will generate the file Views\Posts\MyEmptyView.cshtml in your project.

##Scaffolding Database Context

You can scaffold extra properties for your database context by using the **DbContext** scaffolder and specifying the model class and the name of the database context class. Create a new Blog class, as shown below.

using System;

    using System;
    
    namespace BloggingSite.Models
    {
        public class Blog
        {
            public int Id { get; set; }
            public string Name { get; set; }
        }
    }

 Now run the command below to update your database context and a new Blogs property will be added to your BloggingSiteContext class.

    Scaffold DbContext Blog BloggingSiteContext

The new BloggingSiteContext class is shown below.

    public class BloggingSiteContext : DbContext
    {
        // You can add custom code to this file. Changes will not be overwritten.
        // 
        // If you want Entity Framework to drop and regenerate your database
        // automatically whenever you change your model schema, add the following
        // code to the Application_Start method in your Global.asax file.
        // Note: this will destroy and re-create your database with every model change.
        // 
        // System.Data.Entity.Database.SetInitializer(
            new System.Data.Entity.DropCreateDatabaseIfModelChanges
                <BloggingSite.Models.BloggingSiteContext>());

    	
        public DbSet<BloggingSite.Models.Post> Posts { get; set; }
    	
        public DbSet<BloggingSite.Models.Blog> Blogs { get; set; }
    }

##Scaffolding Controller Actions

You can scaffold new controller actions by using the **Action** scaffolder and specifying the name of the controller, without the Controller part, and the name of the new action method.

    Scaffold Action Posts MyAction

This will add a new MyAction method at the end of your PostsController class.

    public ViewResult MyAction()
    {
        return View();
    }

It will also create an associated view.

You can also scaffold actions that can have data posted to them. If you run the following command a new action will be created that takes a posted Blog object.

    Scaffold Action Posts MyPostAction -ViewModel Blog -Post

The PostsController will now have a new MyPostAction method at the end.

	[HttpPost, ActionName("MyPostAction")]
	public ActionResult MyPostActionPost(Blog blog)
	{
		if (ModelState.IsValid) {
			return RedirectToAction("Index");
		} else {
			return View(blog);
		}
	}

That is the end of our introduction to using MVC Scaffolding with SharpDevelop.

##Modifications Made to Original MVC Scaffolding

1. Mvc Scaffolding projects now compiled against SharpDevelop's Package Management assembly which implements the Visual Studio API.
2. T4 template generation now uses [MonoDevelop's T4 Templating Engine](http://community.sharpdevelop.net/blogs/mattward/archive/2011/06/26/T4TemplatesInSharpDevelop.aspx).
3. T4Scaffolding.SharpDevelop.1.0.8 NuGet package depends on EntityFramework.SharpDevelop NuGet package instead of the original EntityFramework NuGet package This allows an upgrade to the NuGet package EntityFramework.SharpDevelop.5.0.0 which supports migrations in SharpDevelop.

Source code for the modified MVC Scaffolding can be found on [CodePlex](http://mvcscaffolding.codeplex.com/SourceControl/network/forks/MattWard/mvcscaffolding).

**[Update 2012-11-10]** Added information on MvcScaffolding.SharpDevelop 1.0.9 NuGet package requiring newer version of SharpDevelop. Updated "Scaffolding Database Context" section with example Blog class. Updated "Modifications Made to Original MVC Scaffolding" section with change made to T4Scaffolding.SharpDevelop NuGet package.





