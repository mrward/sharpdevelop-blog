# Intro

The SharpDevelop team recently announced for SharpDevelop 5 a new [Addin Manager](http://community.sharpdevelop.net/blogs/andreasweizel/archive/2013/06/10/introducing-the-new-addin-manager-in-sharpdevelop-5.aspx) and an [online gallery for these addins](http://www.myget.org/gallery/sharpdevelop) which is hosted on MyGet.

Addins extensions for SharpDevelop 5 are published as NuGet packages to a MyGet feed. NuGet gives SharpDevelop all the benefits of versioning and updating addins whilst MyGet provides a central place where SharpDevelop addins can be found and downloaded from.

The MyGet feed for SharpDevelop is a community feed which is open to anyone to publish their own addins.

SharpDevelop has been extensible through addins from the very beginning but it has never before had an online gallery of addins. The majority of IDEs have a way to find and install extensions using an online gallery. WebMatrix is one example that provides extensions through from its own [NuGet feed](http://extensions.webmatrix.com/) just like SharpDevelop.

# Background

About a year ago there was a discussion on the SharpDevelop mailing list about having an online source of addins for SharpDevelop. [Andreas Weizel](http://community.sharpdevelop.net/blogs/andreasweizel) had started work on a way to obtain addIns from an online source. At the time it was not based on NuGet. [Christoph Wille](http://community.sharpdevelop.net/blogs/christophwille/), Project Manager for SharpDevelop, suggested looking at re-using the NuGet gallery for hosting the packages and NuGet for downloading and installing addins.

One idea was to put the addins on the main NuGet.org feed and add a special tag to them to distinguish them. We realised that it made much more sense for SharpDevelop to have its own gallery where addins can be searched for and installed from. Whilst we could have created a server and hosted a NuGet gallery ourselves it was much more simple to use MyGet.

Now let us take a look at how you can make your own addins available for SharpDevelop 5.

# Publishing an Addin with MyGet

If you have published a NuGet package to the gallery hosted on NuGet.org then you will have no problem publishing an addin to the SharpDevelop gallery. Here we will look at the steps taken to publish the Mono addin to the MyGet gallery. The [source code](https://github.com/icsharpcode/SharpDevelop/tree/newNR/samples/Mono) for this addin is available as a sample on GitHub. With the Mono addin compiled we need to first create a [.nuspec file](http://docs.nuget.org/docs/reference/nuspec-reference) that will define the NuGet package that will be published to the MyGet gallery.

		<?xml version="1.0"?>
		<package xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
			<metadata xmlns="http://schemas.microsoft.com/packaging/2010/07/nuspec.xsd">
				<id>Mono</id>
				<version>1.0</version>
				<authors>Matt Ward</authors>
				<owners>SharpDevelop</owners>
				<requireLicenseAcceptance>false</requireLicenseAcceptance>
				<description>Mono support for SharpDevelop</description>
				<iconUrl>http://community.sharpdevelop.net/blogs/mattward/SharpDevelop.png</iconUrl>
				<summary>Mono support for SharpDevelop</summary>
				<releaseNotes></releaseNotes>
				<language>en-US</language>
				<tags>mono</tags>
			</metadata>
			<files>
				<file src="..\..\..\AddIns\Samples\Mono.Addin\**\*.*" target="" />
			</files>
		</package>

In the above .nuspec file we are adding all the addin files, including any subdirectories, by using the double wildcard ** to save us from explicitly specifying every file. SharpDevelop requires the main addin assembly and its associated .addin configuration file to be in the root of the NuGet package. This is why the target defined in the .nuspec file has been left empty.

Now we can generate the NuGet package by running NuGet.exe from the command line.

    nuget pack mono.nuspec
    
You will see a "Assembly outside lib folder" warning which is can be ignored since we are not going to be adding this NuGet package to a project. We have now generated the Mono.1.0.nupkg file which is what we will publish to MyGet.

To publish to MyGet you will need to create a MyGet account and use your personal API key. Your API key can be found under Account Settings in the Security section. Copy your API key and use it to replace **Your-API-Key** the following command line.

    nuget push Mono.1.0.nupkg Your-API-Key -Source https://www.myget.org/F/sharpdevelop/api/v2/package

Running the above command will then publish the Mono addin to MyGet.

Now anybody using SharpDevelop 5 can install the Mono addin by selecting AddIn Manager from the Tools menu.

![AddIn Manager dialog](AddInManagerDialog.png)

Select Available to show the list of addins on the MyGet gallery. Select the Mono Addin and click the Install button. After restarting SharpDevelop the addin will then be available and ready to use.


# Bio

Matt Ward is .NET developer at [Pebble Code](http://pebblecode.com/). In his spare time he works on SharpDevelop, which he has been involved with since 2004, and more recently has been working on a [NuGet addin for MonoDevelop and Xamarin Studio](https://github.com/mrward/monodevelop-nuget-addin).

Blog: [community.sharpdevelop.net/blogs/mattward](http://community.sharpdevelop.net/blogs/mattward)
Twitter: [@sharpdevelop](http://twitter.com/sharpdevelop)


# SharpDevelop Bio

[SharpDevelop](http://www.icsharpcode.net/OpenSource/SD/) is an Integrated Development Environment (IDE) for .NET applications created back in 2000 by [Mike Krüger](http://mikemdblog.blogspot.co.uk/). It supports the development of applications written in C#, Visual Basic.NET, F# and IronPython. It is [open source](http://github.com/icsharpcode/sharpdevelop) and written in C#. 