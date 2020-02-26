======================
Command Line Interface
======================

The TypeGen Command Line Interface (CLI) can be used either from the Package Manager Console, after installing TypeGen `NuGet package <https://www.nuget.org/packages/TypeGen>`_, or as a `.NET CLI tool <https://www.nuget.org/packages/dotnet-typegen>`_.

.. container:: Note

    You may need to **restart Visual Studio** before you can start using TypeGen in the Package Manager Console


.. container:: Note

    **Note: This issue has been resolved in Visual Studio 2017 RC**
	
    There is a known issue with using TypeGen from Package Manager Console with ASP.NET Core projects, **under Visual Studio 2015**: Package Manager Console doesn't see TypeGen.ps1. If this happens to you, you can copy TypeGen.ps1 from your NuGet packages folder to your solution/project directory and use it from PowerShell or CMD. Alternatively, you may also consider using `TypeGen .NET CLI tool <https://www.nuget.org/packages/dotnet-typegen>`_ instead of TypeGen Package Manager Console tool.


After running the *[dotnet-]typegen* command, the following set of actions is performed for each of the specified project folders:

#. The CLI reads the assembly file (.dll or .exe). By default (if no assembly path is specified in the config), the assembly file is searched recursively in the project folder's *bin* directory. The name of the assembly must match the name of the .csproj or .xproj file present in the project folder.

#. File generation is performed based on the options specified in the configuration file

New syntax (>= 2.0.0)
=====================

.. code-block:: text

	[dotnet-]typegen [options] [command]
	
	Options:
	-h|--help               Show help information
	-v|--verbose            Show verbose output
	-p|--project-folder     Set project folder path(s)
	-c|--config-path        Set config path(s) to use
	
	Commands:
	generate     Generate TypeScript files
	getcwd       Get current working directory

Old syntax (1.x.x)
==================
	
.. code-block:: text

	TypeGen ProjectFolder1[|ProjectFolder2|(...)] [-Config-Path path1[|path2|(...)]] [Get-Cwd] [-h | -Help] [-v | -Verbose]

**Arguments/options**

========================  ======  
ProjectFolderN            The path to project(s) that TypeScript sources will be generated for. Package Manager Console runs from the solution directory by default, so usually ProjectFolderN would be the name of the project.

-Config-Path              Paths to config files for the specified projects (the order of config paths must match the order of projects). If a project doesn't have its config specified, *tgconfig.json* file from ProjectFolderN will be used. If *tgconfig.json* is not present, the default configuration will be used.

Get-Cwd                   A utility option. If present, the current working directory of the CLI will be outputted. No file generation will occur when this option is present.

-h or -Help               Shows the help.

-v or -Verbose            The CLI will run in verbose mode. More information will be outputted.
========================  ======

Configuration file
==================

TypeGen CLI uses a JSON configuration file to read the file generation options. By default, the configuration file is read from the *tgconfig.json* file present in the specified project folder. If *tgconfig.json* does not exist, default options are used. Configuration file path can also be specified as a CLI argument (see syntax above). For configuration parameters not present in the configuration file, their default values are used upon file generation.

The table below shows all available config parameters, with their default values and a description:

====================================== =================== ====================================== ===================
Parameter                              Type                Default value                          Description
====================================== =================== ====================================== ===================
addFilesToProject                      boolean             false                                  **Only for .NET Framework apps (not .NET Core)**. Whether to add the generated TypeScript files to the project file (\*.csproj)

assemblies                             string[]            []                                     An array of paths to assembly files to generate TypeScript sources from. If null or empty, the default strategy for finding an assembly will be used. **Note:** the order of assemblies also determines the order of reading any custom converters.

**(DEPR.)** assemblyPath               string              null                                   The path to the assembly file with C# types to generate TypeScript sources for. If null or empty, the default strategy for finding an assembly will be used.

buildProject                           boolean             false                                  Whether to build the project before performing file generation

clearOutputDirectory                   boolean             false                                  Whether to clear the output directory before generating new files (removing all files and recursively removing all subdirectories in the output directory)

**(DEPR.)** createIndexFile            boolean             false                                  Whether to generate an index (barrel) file in the root TypeScript output directory. The generated barrel file exports everything from all generated TypeScript files. This option should be avoided in favor of generating barrels from generation specs.

csNullableTranslation                  string              ""                                     Determines which strict-null-checking type unions will be added to C# nullable property types by default. Possible values: "null", "undefined", "null|undefined" or "".

customTypeMappings                     Object              {}                                     Object containing a map of custom [C# to TypeScript] type mappings (example below)

defaultValuesForTypes                  Object              {}                                     Object containing a map of default values for the specified TypeScript types (example below)

enumStringInitializers                 boolean             false                                  Whether to use TypeScript enum string initializers by default

enumStringInitializersConverters (*)   string[]            []                                     Converter chain used for converting C# enum value names to TypeScript enum string initializers. See the (*) explanation below regarding ways in which class names can be specified.

enumValueNameConverters (*)            string[]            []                                     Converter chain used for converting C# enum value names to TypeScript enum value names. See the (*) explanation below regarding ways in which class names can be specified.

explicitPublicAccessor                 boolean             false                                  Whether to use explicit *public* accessor in the generated TypeScript class files

externalAssemblyPaths                  string[]            []                                     An array of paths to external assemblies. These paths are searched (recursively) for any assembly references that cannot be automatically resolved. NuGet package folders (global + machine-wide and project fallback) are searched by default.

fileHeading                            string              null                                   TypeScript file heading text (default is "(...) This is a TypeGen auto-generated file. (...)")

fileNameConverters (*)                 string[]            ["PascalCaseToKebabCaseConverter"]     Converter chain used for converting C# type names to TypeScript file names. See the (*) explanation below regarding ways in which class names can be specified.

generateFromAssemblies                 boolean             null                                   Whether to generate files from assemblies specified in `assemblies` parameter. If null, files are generated from assemblies only if no generation specs are specified.

generationSpecs (*)                    string[]            []                                     An array of generation specs to be used for file generation. See the (*) explanation below regarding ways in which class names can be specified.

outputPath                             string              ""                                     Output path for generated files, relative to the project folder.

propertyNameConverters (*)             string[]            ["PascalCaseToCamelCaseConverter"]     Converter chain used for converting C# property/field names to TypeScript property names. See the (*) explanation below regarding ways in which class names can be specified.

singleQuotes                           boolean             false                                  Whether to use single quotes for string literals in the generated TypeScript files

tabLength                              number              4                                      The number of spaces per tab in the generated TypeScript files

typeNameConverters (*)                 string[]            []                                     Converter chain used for converting C# type names to TypeScript type names. See the (*) explanation below regarding ways in which class names can be specified.

typeScriptFileExtension                string              "ts"                                   File extension for the generated TypeScript files

typeUnionsForTypes                     Object              {}                                     Object containing a map of [TypeScript type -> type unions] mappings (example below)

useDefaultExport                       boolean             false                                  Whether to use TypeScript default exports by default

useTabCharacter                        boolean             false                                  Whether to use the tab character instead or multiple spaces
====================================== =================== ====================================== ===================

(*) The rules for specifying class names are as follows:

* Class names can be specified as a name or a fully qualified name.

* If only the name of a class is specified, the class will first be searched in the assemblies specified in *assemblies* (or the project's assembly if no assemblies are specified) and then (if not found) in *TypeGen.Core*.

* To read a class from a specific assembly, path can be defined in the following format: *assembly/path/assembly.dll:ClassName*, where assembly path is relative to the project's folder.

Example
-------

An example of a configuration file (*tgconfig.json*) is presented below:

.. code-block:: json

	{
	    "assemblies": ["my/app/MyApp.Web.dll", "my/app/MyApp.Models.dll"],
	    "fileNameConverters": ["converters/MyApp.Converters.dll:StripDto", "PascalCaseToKebabCase"],
	    "typeNameConverters": ["converters/MyApp.Converters.dll:Fqcn.Converters.StripDto"],
	    "propertyNameConverters": [],
	    "enumValueNameConverters": ["UnderscoreCaseToPascalCase"],
	    "typeScriptFileExtension": "ts",
	    "tabLength": 2,
	    "explicitPublicAccessor": true,
	    "defaultValuesForTypes": {
	        "number": "-1",
	        "Date | null": "null",
	        "string": "\"\""
	    },
	    "customTypeMappings": {
	        "System.DateTime": "string",
	        "Some.Custom.Type": "number"
	    },
	    "typeUnionsForTypes": {
	        "string": ["null", "undefined"],
	        "Date": ["string"]
	    }
	}