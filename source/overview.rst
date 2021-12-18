========
Overview
========

About TypeGen
=============

TypeGen is a command line tool that generates single-class-per-file TypeScript sources from C#. Its primary use is for keeping TypeScript and C# DTOs in sync, and for this reason not all C# features are translated to TypeScript (for example methods are ignored).

Requirements
============

For .NET Standard compatibility, see `compatibility table <https://docs.microsoft.com/en-us/dotnet/articles/standard/library>`_.

**Versions >= 2.5.0**

* CLI: .NET 5.0
* TypeGen.Core (and programmatical API): .NET Standard 2.0, .NET 5.0

**Versions 2.0.0 - 2.4.9**

* CLI (when used from Package Manager console): .NET Core 3.1
* CLI (when used as a .NET tool): .NET Core 2.1, .NET Core 2.2, .NET Core 3.0, .NET Core 3.1
* Programmatical API: .NET Standard 1.3, .NET Standard 2.0

**Versions 1.5.7 - 1.6.7**

* CLI: .NET Core 2.0
* Programmatical API: .NET Standard versions: 1.3 and 2.0

**Versions 1.5.0 - 1.5.6**

* CLI: .NET Framework 4.6
* Programmatical API: .NET Standard 1.3

**Versions <= 1.4.x**

.NET Framework 4.0

Getting started
===============

To generate the sources, first the C# types to translate should be selected, and then the TypeGen command should be run to create the TypeScript files.
By default the generated source files are saved to the file system, but this can be customised using the programmatical API (see the :doc:`Programmatical API section <programmaticalapi>`).

Installation
------------

To install TypeGen, add the `TypeGen NuGet package <https://www.nuget.org/packages/TypeGen>`_ to your project. After adding this package, the :code:`TypeGen` command will be available in the Package Manager console.

TypeGen is also available as a .NET tool, using `this package <https://nuget.org/packages/dotnet-typegen>`_.

Selecting C# types to generate
------------------------------

C# types to generate can be selected in 2 ways:

1. By specifying :doc:`attributes <attributes>` on the C# types to generate.

2. By creating one or more specification files (called :doc:`generation specs <generationspec>` - a generation spec is a C# class specifying which types to generate) anywhere in your project. To configure TypeGen to use a generation spec, create a `tgconfig.json` file in the project's root directory with the following content:

.. code-block:: json

    {
        "generationSpecs": ["MyGenerationSpec"]
    }

After adding attributes or generation specs, the project should be built to run the :code:`TypeGen` command.

Running the TypeGen command
---------------------------

To run TypeGen from the Package Manager console: open the PM console, select your project from the dropdown list, and then type :code:`TypeGen generate` or :code:`TypeGen -p "MyProjectName" generate` (depending on the current working directory of the PM Console; you might have to restart Visual Studio).

To run TypeGen as a .NET tool, type :code:`dotnet-typegen generate` or :code:`dotnet-typegen -p "MyProjectName" generate` (depending on the current working directory) in the system console.

Example
=======

For example, to translate a C# class named *ProductDto*:

1. When using attributes, the class should be marked:

.. code-block:: csharp

    [ExportTsClass]
    public class ProductDto
    {
        public decimal Price { get; set; }
        public string[] Tags { get; set; }
    }
	
2. When using "generation specs", the class should be listed in a GenerationSpec file:

.. code-block:: csharp

    public class MyGenerationSpec : GenerationSpec
    {
        public MyGenerationSpec()
        {
            AddClass<ProductDto>();
        }
    }

When using generation specs, a `tgconfig.json` file should also be created directly in the project's root directory with the following content:

.. code-block:: json

    {
        "generationSpecs": ["MyGenerationSpec"]
    }

After finishing 1. or 2., build the project and type :code:`TypeGen generate` or :code:`TypeGen -p "MyProjectName" generate` (depending on the current working directory of the PM Console) into the Package Manager Console (you might have to restart Visual Studio). For the .NET tool, type :code:`dotnet-typegen generate` in the system console.

A single TypeScript file named *product-dto.ts* should be created in the project's root directory with the following content:

.. code-block:: typescript

	export class ProductDto {
	    price: number;
	    tags: string[];
	}
