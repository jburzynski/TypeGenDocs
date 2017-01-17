====================
Programmatical usage
====================

If the :doc:`Command Line Interface <cli>` doesn't meet your requirements, or you just want to generate TypeScript files from the code level, you can use the TypeGen file generator class, located in TypeGen.Core assembly.

The Generator class
===================

After installing TypeGen as a `NuGet package <https://www.nuget.org/packages/TypeGen>`_, you'll have access to the *Generator* (*TypeGen.Core.Generator*) class, which is responsible for generating TypeScript files. This class is also used by :doc:`TypeGen CLI <cli>` for this purpose.

The class itself has 1 public method *Generate*, in 2 versions:

* *Generate(Assembly assembly)*
* *Generate(Type type)*

The first version serves for generating files for all annotated types in an assembly (this is the overload used by the CLI). The second one is used for generating TypeScript file(s) for a specific C# type. The generated files are immediately saved to the file system.

Unit testing
============

It's possible to mock the *Generator* class when performing unit tests. The class implements an interface *IGenerator*, which can be used for creating the *Generator* class mocks.

Generator options
=================

The *Generator* class also has 1 public property named *Options*, which holds the file generation options. The options can be assigned by setting the *Options* property to an instance of the *GeneratorOptions* class.

The *GeneratorOptions* class has the following properties (generation options):

* **FileNameConverters** : *TypeNameConverterCollection* - a collection (chain) of converters used for converting C# file names to TypeScript file names. Default is PascalCase to kebab-case.
* **TypeNameConverters** : *TypeNameConverterCollection* - a collection (chain) of converters used for converting C# type names (classes, enums etc.) to TypeScript type names. Default is empty (preserves original type name).
* **PropertyNameConverters** : *NameConverterCollection* - a collection (chain) of converters used for converting C# class property names to TypeScript class property names. Default is PascalCase to camelCase.
* **EnumValueNameConverters** : *NameConverterCollection* - a collection (chain) of converters used for converting C# enum value names to TypeScript enum value names. Default is empty (preserves original name).
* **ExplicitPublicAccessor** : *bool* - whether to generate explicit "public" accessor in TypeScript classes. Default is *false*.
* **TypeScriptFileExtension** : *string* - file extension used for the generated TypeScript files. Default is "ts".
* **TabLength** : *int* - number of space characters per tab. Default is 4.
* **SingleQuotes** : *bool* - whether to use single quotes for string literals in generated TypeScript files. Default is *false*.
* **BaseOutputDirectory** : *string* - the base directory for generating TypeScript files. Any relative paths defined in the *ExportTs...* attributes (as *OutputDir*) will be resolved relatively to this path.

Example
=======

An example of programmatical usage is shown below:

.. code-block:: csharp

	var options = new GeneratorOptions { BaseOutputDirectory = @"C:\my\output" }; // create the options object
	var generator = new Generator { Options = options }; // create the generator instance
	var assembly = Assembly.GetCallingAssembly(); // get the assembly to generate files for
	generator.Generate(assembly); // generates the files