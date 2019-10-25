========
Overview
========

About TypeGen
=============

TypeGen is a command line tool that generates single-class-per-file TypeScript sources from C#. Its primary use case is keeping TypeScript model classes/interfaces in sync with C# models, and for this reason not all C# features are translated to TypeScript (e.g. methods or events will be ignored when generating TypeScript sources).

Requirements
============

For .NET Standard compatibility, see `compatibility table <https://docs.microsoft.com/en-us/dotnet/articles/standard/library>`_.

**Versions >= 2.0.0**

* CLI: .NET Core 2.1+ (.NET Core version for the CLI is updated whenever a new version of .NET Core is released)
* TypeGen.Core: .NET Standard versions: 1.3 and 2.0

**Versions 1.5.7 - 1.6.7**

* CLI: .NET Core 2.0
* TypeGen.Core: .NET Standard versions: 1.3 and 2.0

**Versions 1.5.0 - 1.5.6**

* CLI: .NET Framework 4.6
* TypeGen.Core: .NET Standard 1.3

**Versions <= 1.4.x**

.NET Framework 4.0

How to use it
=============

The general idea is: first you select C# types to be generated to TypeScript, then you run the :code:`typegen` command and the TypeScript files are generated.

The details are covered in the following subsections.

Installing TypeGen
------------------

To install TypeGen, add the `TypeGen NuGet package <https://www.nuget.org/packages/TypeGen>`_ to your project. After adding this package, the :code:`TypeGen` command will be available in the Package Manager console.

You can also use TypeGen as a .NET Core CLI tool, in which case you should install it from `this package <https://nuget.org/packages/TypeGen.DotNetCli>`_.

Selecting C# types to generate
------------------------------

C# types to generate can be selected in 2 ways:

1. By specifying :doc:`attributes <attributes>` on the C# types you wish to generate.

2. By creating one or more :doc:`generation specs <generationspec>` (a generation spec is a C# class which specifies which types should be generated) somewhere in your project. To instruct TypeGen to use your generation spec, place this content in a file named `tgconfig.json` in your project directory:

.. code-block:: json

    {
        "generationSpecs": ["MyGenerationSpec"]
    }

Attributes offer quicker, but more functionally-restricted way of selecting types to generate (recommended for smaller projects), whereas generation specs require more initial work to do, but offer richer functionality (recommended for bigger projects).

After adding attributes or creating/changing generation spec(s), you need to build your project before running the :code:`TypeGen` command.

Running the TypeGen command
---------------------------

To run TypeGen from the Package Manager console, open the PM console, select your project from the dropdown list and type :code:`TypeGen generate` or :code:`TypeGen -p "MyProjectName" generate` (depending on the current working directory of the PM Console; you might have to restart Visual Studio).

To run TypeGen as a .NET Core CLI tool, type :code:`dotnet-typegen generate` or :code:`dotnet-typegen -p "MyProjectName" generate` (depending on your current location) in your OS command shell.

Example
=======

Let's say you have a *ProductDto* class that you want to export to TypeScript.

1. If you're using attributes, annotate your class with an appropriate attribute:

.. code-block:: csharp

    [ExportTsClass]
    public class ProductDto
    {
        public decimal Price { get; set; }
        public string[] Tags { get; set; }
    }
	
2. If you're using a generation spec, first create your generation spec somewhere in your project:

.. code-block:: csharp

    public class MyGenerationSpec : GenerationSpec
    {
        public MyGenerationSpec()
        {
            AddClass<ProductDto>();
        }
    }

...and then create a file named `tgconfig.json` directly in your project folder and place the following content in this file:

.. code-block:: json

    {
        "generationSpecs": ["MyGenerationSpec"]
    }

After finishing instructions described in either 1. or 2., **build your project** and type :code:`TypeGen generate` or :code:`TypeGen -p "MyProjectName" generate` (depending on the current working directory of the PM Console) into the Package Manager Console (you might have to restart Visual Studio). Instead of using the Package Manager Console, you can also use TypeGen as a .NET Core CLI tool by typing :code:`dotnet-typegen generate` in your OS command shell.

This will generate a single TypeScript file (named *product-dto.ts*) in your project directory. The file will look like this:

.. code-block:: typescript

	export class ProductDto {
	    price: number;
	    tags: string[];
	}

What next
=========

More details about the available configuration options (that you can place in `tgconfig.json`) are described in the :doc:`CLI <cli>` section. You can also find out more about :doc:`attributes <attributes>` or :doc:`generation specs <generationspec>` in their dedicated sections.

If you need to convert between different naming conventions (i.e. your C# code uses different conventions than your TypeScript code), you can utilize the :doc:`converters <converters>` functionality.

Instead of using the :code:`TypeGen` command in the console, you can generate files directly from your code using the :doc:`TypeGen programmatical API <programmaticalapi>`.
