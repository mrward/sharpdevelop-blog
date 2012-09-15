[SharpDevelop 4.3](http://build.sharpdevelop.net/buildartefacts/#SDMAIN) now has support for  [T4MVC](http://t4mvc.codeplex.com/).

[T4MVC](http://t4mvc.codeplex.com/) is a set of T4 templates, created by [David Ebbo](http://blog.davidebbo.com/), that will generate strongly typed helpers for an ASP.NET MVC application. It will allow you to remove strings from your MVC application making your application easier to maintain.

So let us take a look at how to use T4MVC with SharpDevelop. First you should open or [create a new ASP.NET MVC application in SharpDevelop](http://community.sharpdevelop.net/blogs/mattward/archive/2012/01/29/AspNetMvcSupport.aspx).

##Installing T4MVC

T4MVC is available as a [NuGet package](http://nuget.org/packages/T4MVC.SharpDevelop). The NuGet package you should download is [T4MVC.SharpDevelop](http://nuget.org/packages/T4MVC.SharpDevelop). This contains a modified version of the original T4MVC template that can be used with SharpDevelop. Further details on all the modifications made to the T4MVC template can be found at the end of this post. Install the T4MVC.SharpDevelop NuGet package either from the [NuGet package management console](http://community.sharpdevelop.net/blogs/mattward/archive/2011/06/05/NuGetPowerShellConsole.aspx) or by using the [Manage Packages dialog](http://community.sharpdevelop.net/blogs/mattward/archive/2011/07/24/NuGet14.aspx).

After installation two new T4 template files will be added to your project.

1. T4MVC.tt - main template that generates the strongly typed helpers.
2. T4MVC.tt.settings.t4 - holds configuration settings used by main T4MVC.tt template.

##Generating T4MVC's Strongly Typed Helpers

To run the T4MVC template manually, select it in the **Projects** window, right click and select **Execute Custom Tool**. This will generate a set of files as dependencies of the T4MVC.tt file.

![T4MVC Generated Files in Projects window](/Images/T4MVCGeneratedFiles.png)

It will also make some modifications to your controller classes.  The T4MVC template will change your controller classes so they are **partial**. It will also change your controller methods so they are **virtual**. What has been changed will be recorded as warnings in the **Errors** window.

![T4MVC modified code warnings in Errors window](/Images/T4MVCWarningsInErrorsWindow.png)

When you make modifications to your application you do not want to have to keep running the T4MVC template manually each time to regenerate the strongly typed helpers. Instead you can configure your project to re-generate the strongly typed helpers on each build.

##Generating T4MVC's Strongly Typed Helpers on each Build

To do this you should open the options for the project. From the **Projects** menu and select **Project Options**. Open the **Custom Tool** tab. In this tab you can choose to run custom tools on each build. In the screenshot below the project is configured to run T4MVC on each build. 

![Project options configured to run T4MVC on every build](/Images/T4MVCCustomToolProjectOptions.png)

If you want to run other custom tools  you can add filenames for items in your project either as a comma separated list or with each file on a separate line. Currently there is no support for wildcards when specifying filenames.

Now that we have generated the strongly typed helpers let us take a look at how to use a selection of them.

##Using T4MVC's Strongly Typed Helpers

###View Names

In your _ViewStart.cshtml you may have a reference to a Razor layout page.

	@{
		Layout = "~/Views/Shared/_Layout.cshtml";
	}

You can replace this string with a strongly typed helper.

	@{
		Layout = MVC.Shared.Views._Layout;
	}

###Action Links

In your views you may be passing action names and controller names as strings [HtmlHelper.ActionLink(string linkText, string actionName, string controllerName)](http://msdn.microsoft.com/en-us/library/dd505070(v=vs.100)).

		<li>@Html.ActionLink("Home", "Index", "Home")</li>
		<li>@Html.ActionLink("Contact", "Contact", "Home")</li>

Instead you can use code that looks like you are calling the controller method.

		<li>@Html.ActionLink("Home", MVC.Home.Index())</li>
		<li>@Html.ActionLink("Contact", MVC.Home.Contact())</li>

###CSS Links

You may be using strings for links, such as for CSS files, in your views.

	<link href="@Url.Content("~/Content/Site.css")" rel="stylesheet" type="text/css">

These can be changed to use the Links.Content helper class.

	<link href="@Links.Content.Site_css" rel="stylesheet" type="text/css">

That is a very quick introduction to T4MVC and covers only a few of the helpers it provides. Further information on T4MVC can be found on the [T4MVC Codeplex site](https://t4mvc.codeplex.com/documentation).

##Changes Made to Original T4MVC Template

1. Visual Studio assembly references have been removed and replaced with a SharpDevelop assembly reference.
2. Visual Studio namespace imports have been removed and replaced with namespace imports for SharpDevelop.
3. EnvDTE.ProjectItem.get_FileNames() replaced with ProjectItem.FileNames() - SharpDevelop currently implements this as a method and not a parameterised property.
4.  EnvDTE.CodeType.get_IsDerivedFrom() replaced with CodeType.IsDerivedFrom() - SharpDevelop currently implements this as a method and not a parameterised property.
5. Removed use of BeginInvoke/EndInvoke which was causing SharpDevelop to hang.

The SharpDevelop specific T4MVC template is maintained in a [repository on github](https://github.com/mrward/t4mvc-sharpdevelop).