
SharpDevelop 4.3 has been updated to use [NUnit 2.6.1](http://nunit.org/index.php?p=releaseNotes&r=2.6.1) which was recently released. The full details on what has changed in NUnit 2.6.1 can be found in the [NUnit release notes](http://nunit.org/index.php?p=releaseNotes&r=2.6.1). One change in this release caused some of SharpDevelop's own unit tests to break so this change is highlighted in the following section. 

##Running unit tests with an STA thread

NUnit 2.6 no longer reads settings from your test configuration file (app.config). This means that you can no longer make NUnit use the STA thread by [setting the default apartment state in your app.config](http://stackoverflow.com/questions/2434067/how-to-run-unit-tests-in-stathread-mode) file. So the following app.config file will not work:

      <configuration> 
        <configSections>
          <sectionGroup name="NUnit">
            <section name="TestRunner"  type="System.Configuration.NameValueSectionHandler" />
            </sectionGroup>
          </configSections>
          <NUnit>
            <TestRunner>
               <!-- The ApartmentState value here is ignored. -->
               <add key="ApartmentState" value="STA" />
            </TestRunner>
          </NUnit>
      </configuration>

Instead you should use the [RequiresSTA attribute](http://www.nunit.org/index.php?p=requiresSTA&r=2.6.1). 

The RequiresSTA attribute can be used at the assembly, class or method level. To make NUnit to run all your unit tests with an STA thread you can add the following code to your AssemblyInfo.cs file in the project containing your unit tests.

    using NUnit.Framework;
    
    [assembly: RequiresSTA]

