
It has been a while since [NuGet support was added to SharpDevelop](http://community.sharpdevelop.net/blogs/mattward/archive/2011/01/23/NuGetSupportInSharpDevelop.aspx) so, after David Fowler announced that he had got [NuGet building under Mono](https://twitter.com/davidfowl/status/276188287721414656), I had a look at porting the SharpDevelop addin over to [MonoDevelop](http://monodevelop.com) and created a [NuGet addin for MonoDevelop](https://github.com/mrward/monodevelop-nuget-addin).

The [addin](https://github.com/mrward/monodevelop-nuget-addin) provides a **Manage Packages dialog** that you can use to add, remove and update NuGet packages in a similar way to Visual Studio or SharpDevelop. Here is a screenshot of the **Manage Packages dialog** open in MonoDevelop running on OpenSuse 12.2

![Manage Packages Dialog with package operation messages](/Images/ManagePackagesDialog.png)

The addin has been tested with the following:

 * MonoDevelop 3.0.5
 * Mono 2.10.9
 * OpenSuse 12.2
 * Windows 7

The addin uses a [slightly customised version of NuGet.Core](https://github.com/mrward/nuget/tree/monodevelop) based on the original [NuGet's source code](http://nuget.codeplex.com/SourceControl/BrowseLatest) available from the mono-build branch. The source code for this customised NuGet.Core is available on [GitHub](https://github.com/mrward/nuget/tree/monodevelop).

## Getting the Addin

The addin's [source code](https://github.com/mrward/monodevelop-nuget-addin) is available on [GitHub](https://github.com/mrward/monodevelop-nuget-addin).

The addin is also provided as a pre-compiled binary available to download from a custom MonoDevelop repository:

[http://mrward.github.com/monodevelop-nuget-addin-repository/3.0.5/main.mrep](http://mrward.github.com/monodevelop-nuget-addin-repository/3.0.5/main.mrep)

You can add this repository to MonoDevelop via the [Add-in Manager](http://monodevelop.com/Documentation/Installing_Add-ins).

Please note that the addin is a beta release and more work is still to be done.

## Manage Packages Dialog

The **Manage Packages Dialog** can be opened from the **Projects** menu by selecting **Manage NuGet Packages**.

It can also be opened by selecting the **Solution** window, right clicking either the solution, the project or the project's references and selecting **Manage NuGet Packages**.

As a NuGet package is installed or uninstalled you can see the package operations that executed by expanding the **Messages** section near the bottom of the dialog.

![Manage Packages Dialog with package operation messages](/Images/ManagePackagesDialogWithMessages.png)

## Configuring Package Sources

NuGet Package Sources can be configured in MonoDevelop's options.

![Configuring NuGet Package Sources in MonoDevelop's Options](/Images/NuGetPackageSourcesOptionsDialog.png)

In Linux you can open the options dialog by selecting **Preferences** from the **Edit** menu. In Windows you can select **Options** from the **Tools** menu. In the Options dialog scroll down the categories on the left hand side until you find **NuGet** and then select **Package Sources**. Here you can add and remove NuGet package sources.

##PowerShell

Any PowerShell scripts in a NuGet package will be ignored by the addin since Mono does not currently have support for them.

There have been some attempts to implement PowerShell on Mono starting with Igor Moochnick who created [Pash](http://pash.sourceforge.net) in 2008. Work on this was picked up [launchpad.net](https://launchpad.net/pash) by Jonathan Ben-Joseph but that work stopped. The latest effort on bringing PowerShell to Mono is Jay Bazuzi with recent work done on the Pash source code available on [GitHub](https://github.com/JayBazuzi/Pash2).

* [Pash on SourceForge](http://pash.sourceforge.net)
* [Pash on LaunchPad](https://launchpad.net/pash)
* [Pash2 on GitHub](https://github.com/JayBazuzi/Pash2)

There was also some [discussion in the NuGet forums](http://nuget.codeplex.com/discussions/246658) on allowing some way of being able to write a [.NET version of the PowerShell scripts](http://nuget.codeplex.com/workitem/720) (install.ps1, uninstall.ps1 and init.ps1).
