## Unity NuGet Package - Implementing EnvDTE.Solution.Globals

Older versions of SharpDevelop 4.3 displayed an error when the Unity NuGet package was installed.  Here we take a look at what the problem was and how it was fixed.

##The Problem

On installing the Unity NuGet package version 2.1.505.2 your solution file will be modified. A new section will be added as shown below:

	GlobalSection(ExtensibilityGlobals) = postSolution
		EnterpriseLibraryConfigurationToolBinariesPath = packages\Unity.2.1.505.2\lib\NET35
	EndGlobalSection

Now this is done with a Powershell script that is included with the NuGet package. First let us take a look at the Unity NuGet package's install script install.ps1

    Import-Module (Join-Path $toolsPath Utils.psm1)

    $relativeInstallPath = Get-RelativePath ([System.Io.Path]::GetDirectoryName($dte.Solution.FullName)) $installPath
    $folder = (Join-Path (Join-Path $relativeInstallPath "lib")  "NET35")
    Add-ToolFolder $folder

So this script imports another script **Utils.psm1** and then creates a string that holds the path to the **lib\NET35** folder where the Unity package is installed. This path will typically be **packages\Unity.2.1.505.2\lib\NET35** and is passed to the **Add-ToolFolder** function.  This function is defined in the **Utils.psm1** file. The Add-ToolFolder uses two other functions **Get-ExtenderPropertyValue** and **Set-ExtenderPropertyValue** which are used to read values defined in your solution and update them.

      function Get-ExtenderPropertyValue
      {
          if( $dte.Solution.Globals.VariableExists("EnterpriseLibraryConfigurationToolBinariesPath") -and $dte.Solution.Globals.VariablePersists("EnterpriseLibraryConfigurationToolBinariesPath") )
          {
              return $dte.Solution.Globals.VariableValue("EnterpriseLibraryConfigurationToolBinariesPath")
          }
          return ""
      }
    
      function Set-ExtenderPropertyValue([System.String] $value)
      {
          if( [System.String]::IsNullOrWhiteSpace($value) )
          {
              if( $dte.Solution.Globals.VariableExists("EnterpriseLibraryConfigurationToolBinariesPath") )
              {
                  $dte.Solution.Globals.VariablePersists("EnterpriseLibraryConfigurationToolBinariesPath") = $False
              }
          }
          else
          {
              $dte.Solution.Globals.VariableValue("EnterpriseLibraryConfigurationToolBinariesPath") = $value
              $dte.Solution.Globals.VariablePersists("EnterpriseLibraryConfigurationToolBinariesPath") = $True
          }
      }

Now the error returned by SharpDevelop was because the $dte.Solution had no Globals property defined. 

##Implementing EnvDTE.Solution.Globals

The [Globals](http://msdn.microsoft.com/en-us/library/4aahce9k(v=vs.100)) COM interface defines several properties of which we only need to implement the following three to get Unity to install without any errors:

 * [VariableExists](http://msdn.microsoft.com/en-us/library/envdte.globals.variableexists(v=vs.100))
 * [VariablePersists](http://msdn.microsoft.com/en-us/library/envdte.globals.variablepersists(v=vs.100))
 * [VariableValue](http://msdn.microsoft.com/en-us/library/envdte.globals.variablevalue(v=vs.100))

First let us take a look at VariableExists. 

##Globals.VariableExists

The definition for this property is:

    bool VariableExists[string Name] { get; }

This looks like an indexer but it has a name. However looking at how it is used from Powershell it behaves like a function.

    $dte.Solution.Globals.VariableExists("EnterpriseLibraryConfigurationToolBinariesPath")
    
Since we are interested in making it work with Powershell with the existing Unity NuGet package we can implement it as a method. A simple implementation just to check that Powershell can call the method in the same way as shown above.

	public class Globals
	{
	    string variableName = "test";
	    
	    public bool VariableExists(string name)
	    {
	        return variableName == name;
	    }
	}

To test this you can open Powershell in a command prompt. Set the execution policy so you can run scripts:

	Set-ExecutionPolicy remotesigned -scope process

Create a globals-test.ps1 script file using your favourite text editor.

	$source = @"
	public class Globals
	{
	    string variableName = "test";
	    
	    public bool VariableExists(string name)
	    {
	        return variableName == name;
	    }
	}
	"@
	
	Add-Type -TypeDefinition $source
	$g = New-Object Globals
	
	Write-Host "globals.VariableExists(""test"")"
	$g.VariableExists('test')
	
	Write-Host "globals.VariableExists(""unknown"")"
	$g.VariableExists("unknown")

Then you can run the script in the Powershell command prompt:

    . .\globals-test.ps1

The output should be:

	globals.VariableExists("test")
	True
	globals.VariableExists("unknown")
	False

Now let us take a look at VariableValue.

##Globals.VariableValue

The definition for VariableValue is:

	Object this[string VariableName] { get; set; }

Now this has both a getter and a setter.  So we need to be able to do both of the following in Powershell:

    $value = $dte.Solution.Globals.VariableValue("EnterpriseLibraryConfigurationToolBinariesPath")
    $dte.Solution.Globals.VariableValue("EnterpriseLibraryConfigurationToolBinariesPath") = $value
 
Now let us try implementing the property as an indexer as the definition above:

    public class Globals
    {
        string variableValue = "test";
    
        public object this[string name]
        {
            get { return variableValue; }
            set { variableValue = value as string; }
        }
    }
    
Now this only allows us to access from Powershell as shown below:
    
    $g = New-Object Globals
    $g["test"] = "value"
    $g["test"]
    $g.Item("test") = "value"
    $g.Item("test")

We do not have a VariableValue property and we also have to use square brackets [] to access the variable value or use the Item name, which is what the C# compiler defines the property name to be.

Here Powershell returns the Globals.Item as a PSParameterizedProperty. This is how Powershell exposes parameterised COM properties and makes the property act like a method so we can use the method syntax to access it.

To fix the VariableValue property we could try to introduce a separate class, example below.

    public class Globals
    {
        VariableValues values = new VariableValues();
        
        public object VariableValue
        {
            get { return values; }
        }
    }
    
    public class VariableValues
    {
        string variableValue = "test";
        
        public object this[string name]
        {
            get { return variableValue; }
            set { variableValue = value as string; }
        }
    }

This does not solve the problem. Now we have lost the ability to call Globals.VariableValue("test") and we are now forced to use square brackets.

Another idea is to use the [IndexerName](http://microsoft.com/en-us/library/system.runtime.compilerservices.indexernameattribute.aspx) attribute to change the name of the indexer to VariableValue:

    public class Globals
    {
        string variableValue = "test";
          
        [System.Runtime.CompilerServices.IndexerName("VariableValue")]
        public object this[string name]
        {
            get { return variableValue; }
            set { variableValue = value as string; }
        }
    }

Then we can use it from Powershell in the way we want to:

    $g.VariableValue("test") = "new-value";
    $g.VariableValue('test')

The VariableValue property here is a PSParameterisedProperty so we can use the method syntax to access the property.

We still have a problem here since we cannot have more than one indexer in a C# class. The following will not compile since we have two members called this.

    public class Globals
    {
        string variableValue = "test";
        bool variablePersists;
          
        [System.Runtime.CompilerServices.IndexerName("VariableValue")]
        public object this[string name]
        {
            get { return variableValue; }
            set { variableValue = value as string; }
        }
        
        [System.Runtime.CompilerServices.IndexerName("VariablePersists")]
        public bool this[string name]
        {
            get { return variablePersists; }
            set { variablePersists = value; }
        }
    }

So we cannot repeat the same steps for the VariablePersists property. C# 4.0[ introduced the ability to be able to call named indexer properties for better interoperability with COM](http://msdn.microsoft.com/en-us/library/ee310208(VS.100).aspx) but you cannot create a C# with multiple named indexer properties. However VB.NET does support them but before we switch to VB.NET let us try a few more things with C#.

##Implementing Globals.VariableValue with a struct

Not possible since we cannot add a new **implicit operator** for object since a struct can already implicitly convert to or from an object.

	public struct VariableValueType
	{
		object value;
		
		public VariableValueType(object value)
		{
			this.value = value;
		}
		
		public static implicit operator Object(VariableValueType variableValue)
		{
			return variableValue.value;
		}
		
		public static implicit operator VariableValueType(object value)
		{
			return new VariableValueType(value);
		}
	}

However even if we look at VariablePersists which uses a bool and we could define a struct we still could not use it. 

	public class Globals
	{
		VariablePersistsType values = new VariablePersistsType();
		
		public VariablePersistsType VariablePersists(string name)
		{
			return values;
		}
	}

	public struct VariablePersistsType
	{
		bool value;
		
		public VariablePersistsType(bool value)
		{
			this.value = value;
		}
		
		public static implicit operator bool(VariablePersistsType variablePersists)
		{
			return variablePersists.value;
		}
		
		public static implicit operator VariablePersistsType(bool value)
		{
			return new VariablePersistsType(value);
		}
	}

Here we get an error when trying to set the VariablePersists.

    Assignment failed because [Globals] doesn't contain a settable property 'VariablePersists()'.

## Implementing Globals.VariableValue with a dynamic object

Powershell does not seem to support accessing properties or methods that are defined with a [DynamicObject](http://msdn.microsoft.com/en-us/library/system.dynamic.dynamicobject(v=vs.100).aspx).

So it looks like VB.NET is the only way to do this.

##Implementing Globals.VariableName in VB.NET

	Public Class Globals
		Private m_variableExists As Boolean
		Private m_variableValue As Object = "value"
		Private m_variablePersists As Boolean
	
		Public Property VariableValue(ByVal name As String) As Object
			Get
				Return m_variableValue
			End Get
			Set
				m_variableValue = value
			End Set
		End Property

		Public Property VariablePersists(VariableName As String) As Boolean
			Get
				Return m_variablePersists
			End Get
			Set
				m_variablePersists = value
			End Set
		End Property

		Public ReadOnly Property VariableExists(name As string) As Boolean
			Get
				Return name = "test"
			End Get
		End Property
	End Class
