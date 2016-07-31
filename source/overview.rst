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

TypeGen runs under .NET 3.5 or higher. No external dependencies are required.

Features
============================

TypeGen consists of two components: a core library (contains logic for file generation) and a Command Line Interface that utilizes the file generation logic. In most cases, the CLI should provide enough functionality to generate TypeScript sources as needed. In more complex cases, there is a possibility of using the Generator class directly from your C# code (the Generator class serves for - you guessed it - TypeScript file generation). For more information about using the CLI, please refer to the :doc:`Command Line Interface <cli>` section. More details on using file generation directly from the C# code can be found in the :doc:`Programmatical usage <programmaticalusage>` section.

Among the TypeGen's features are:

* generating TypeScript classes, interfaces and enums - single class per file
* support for collection (or nested collection) property types
* possibility of specifying an output path to generate a TypeScript file to (per class/enum)
* automatic generation of properties' types (if a type is not natively available in TypeScript)
* customizable convertion between C#/TypeScript names (naming conventions)

Getting started
===============

The easiest way to install TypeGen is `via NuGet <https://www.nuget.org/packages/TypeGen>`. After installation, both CLI and core functionality will be available.

Quick example
=============

Let's say you have a *ProductDto* class that you want to export to TypeScript. You first need to mark the class as exportable to TypeScript:

.. code-block:: c#

using TypeGen.Core.TypeAnnotations;

[ExportTsClass]
public class ProductDto
{
    public decimal Price { get; set; }
    public string[] Tags { get; set; }
}

After building your project, type into the Package Manager Console:

.. code-block::

TypeGen ProjectFolder

...where *ProjectFolder* is your project folder, relative to your solution directory.

This will generate a single TypeScript file (named *product-dto.ts*) in your project directory. The file will look like this:

.. code-block:: typescript

export class ProductDto {
    price: number;
    tags: string[];
}

Of course, there are much more things you can do with TypeGen. To find out more, please click *next* or visit a relevant section.