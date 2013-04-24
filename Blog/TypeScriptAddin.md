SharpDevelop 4.3.1 now has support for [TypeScript](http://www.typescriptlang.org/) with an early beta release of a new [Addin](https://github.com/mrward/typescript-addin). 

The Addin glues together the TypeScript language services, which provide all the features needed for code completion and refactoring, and SharpDevelop using [Javascript.NET](http://javascriptdotnet.codeplex.com/) as the bridge between them. Javascript.NET allows SharpDevelop to host Google's V8 JavaScript engine and have JavaScript code interact with .NET objects.

## Features

 * TypeScript compilation on save or build.
 * Code folding.
 * Code completion
 * Find references
 * Rename refactoring
 * Go to definition
 * Quick class browser support.
 * TypeScript syntax highlighting
 
The addin supports:

 * SharpDevelop 4.3.1
 * TypeScript 0.8.3.1

Let us take a quick look at some of these features.

## Code Completion

Code completion works when you press the dot character.

![TypeScript dot code completion](/Images/TypeScriptDotCodeCompletion.png)

Code completion also works when you type the first bracket of a function.

![TypeScript function code completion](/Images/TypeScriptFunctionCompletion.png)

## Find References, Go to Definition and Rename

These menu options can be found by right clicking on a class or a class member.

![TypeScript right click menu options](/Images/TypeScriptTextEditorContextMenu.png)

Go To TypeScript Definition will show the corresponding type definition. Find TypeScript References will show the reference locations in the Output window. Rename will open a dialog box where you can type in the new name and click OK to have it updated everywhere it is referenced.

## Getting the Addin

The addin's [source code](https://github.com/mrward/typescript-addin) is available on [GitHub](https://github.com/mrward/typescript-addin). A [zip file is available to download](http://community.sharpdevelop.net/blogs/mattward/TypeScript/TypeScriptAddin-0.1.zip) containing the pre-compiled addin. Simply unzip the files and install into SharpDevelop using the Addin Manager. The Addin Manager is available by selecting Addin Manager from the Tools menu. Install the addin by clicking the Install Addin button and then restart SharpDevelop.

## Configuring TypeScript Compiler Options

The compiler options are available from the Options dialog (Tools - Options) under the Text Editor category. Here you can change when the compiler is run and what options are passed to the compiler when generating JavaScript code.

![Configuring TypeScript Compiler Options](/Images/TypeScriptCompilerOptionsDialog.png)

## Limitations and Known Issues

 * Limitation with multiple projects using TypeScript files in the same solution. All files are included in the same scope.
 * Performance problems - currently doing work on the UI thread and also re-parsing all TypeScript files each time when a code completion action is triggered.
 * Different behaviour to Visual Studio with classes show in the code completion window. Visual Studio only shows code completion for TypeScript files you have explicitly referenced in a comment. SharpDevelop shows everything in the current project.
 * Unable to easily replace the existing SharpDevelop menu options such as Find References so the standard shortcuts will not work.
 * No support for hovering over a type with the mouse and seeing a tooltip with information about the type.
 
