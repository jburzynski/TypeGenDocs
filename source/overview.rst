========
Overview
========

About TypeGen
=============

TypeGen is a file generator that creates TypeScript source files based on C# classes or enums. The main goal of TypeGen is to allow for single-class-per-file generation for TypeScript sources.

There are many other tools that allow for converting C# classes or enums to their TypeScript counterparts. Such automatic source generation is very helpful when there is a need of keeping the same DTO classes for both C# and TypeScript, for example when consuming a web API. Most of the existing tools, however, allow only for generating multiple types per TypeScript file (grouped in either a module or a namespace). This is not ideal when a directory structure of TypeScript files needs to be preserved - for example when using a TypeScript-based framework.

The main goal of TypeGen is therefore to allow for single-class-per-file generation, which makes it useful when generating TypeScript sources with a defined directory structure.

Requirements
============

For .NET Standard compatibility, see `compatibility table <https://docs.microsoft.com/en-us/dotnet/articles/standard/library>`_.

**Versions >= 1.5.7**

* CLI: .NET Core 2.0
* TypeGen.Core: .NET Standard versions: 1.3 and 2.0

**Versions 1.5.0 - 1.5.6**

* CLI: .NET Framework 4.6
* TypeGen.Core: .NET Standard 1.3

**Versions <= 1.4.x**

.NET Framework 4.0

Features
========

TypeGen consists of two components: a core library (contains logic for file generation) and a Command Line Interface that utilizes the file generation logic. In most cases, the CLI should provide enough functionality to generate TypeScript sources as needed. In more complex cases, there is a possibility of using the core library file generation logic directly from your C# code. For more information about using the CLI, please refer to the :doc:`Command Line Interface <cli>` section. More details on using file generation directly from the C# code can be found in the :doc:`Programmatical usage <programmaticalusage>` section.

Among the TypeGen's features are:

* generating TypeScript classes, interfaces and enums - single class per file
* support for collection (or nested collection) property types
* generic classes/types generation
* support for inheritance
* customizable convertion between C#/TypeScript names (naming conventions)

How it works
============

The big picture is best illustrated by this diagram:

.. image:: images/big-picture.png

In short, TypeGen gets all C# classes/enums (annotated with appropriate attributes) from an assembly and generates corresponding TypeScript files for them in the file system. To find out more, please keep reading this documentation.

Getting started
===============

To use TypeGen in your code or from the PM console, install the `NuGet package <https://www.nuget.org/packages/TypeGen>`_. After installation, both CLI and programmatical API will be available.

If you just want to have TypeGen CLI installed on your computer (so you can use it from Windows command line), you can download the `Chocolatey package <https://chocolatey.org/packages/TypeGen>`_.

Quick example
=============

Let's say you have a *ProductDto* class that you want to export to TypeScript. You first need to mark the class as exportable to TypeScript:

.. code-block:: csharp

	using TypeGen.Core.TypeAnnotations;

	[ExportTsClass]
	public class ProductDto
	{
	    public decimal Price { get; set; }
	    public string[] Tags { get; set; }
	}

After building your project, type into the Package Manager Console:

.. code-block:: text

	TypeGen ProjectFolder

...where *ProjectFolder* is your project folder, relative to your solution directory.

This will generate a single TypeScript file (named *product-dto.ts*) in your project directory. The file will look like this:

.. code-block:: typescript

	export class ProductDto {
	    price: number;
	    tags: string[];
	}

Of course, there are much more things you can do with TypeGen. To find out more, please click *next* or visit a relevant section.