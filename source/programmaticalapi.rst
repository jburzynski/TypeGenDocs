==================
Programmatical API
==================

If the :doc:`Command Line Interface <cli>` doesn't meet your requirements, or you just want to generate TypeScript files from the code level, you can use the TypeGen file generator class, located in TypeGen.Core assembly.

The Generator class
===================

After installing TypeGen as a `NuGet package <https://www.nuget.org/packages/TypeGen>`_, you'll have access to the *Generator* (*TypeGen.Core.Generator*) class, which is responsible for generating TypeScript files. This class is also used by :doc:`TypeGen CLI <cli>` for this purpose.

The class itself has 1 public method *Generate*, in 4 versions:

* *Generate(GenerationSpec generationSpec)*
* *Generate(IEnumerable<Assembly> assemblies)*
* *Generate(Assembly assembly)*
* *Generate(Type type)*

The first overload generates TS sources from a generation spec. The next two overloads serve for generating files for all annotated types in an assembly/assemblies (these overloads are used by the CLI). The last one is used for generating TypeScript file(s) for a specific C# type. The generated files are immediately saved to the file system.

Generator options
=================

The *Generator* class also has 1 public property named *Options*, which holds the file generation options. The options can be assigned by setting the *Options* property to an instance of *GeneratorOptions* class.

Current list of options available in *GeneratorOptions* is provided in `API Reference | GeneratorOptions <http://jburzynski.net/TypeGen/api-reference/api/TypeGen.Core.GeneratorOptions.html>`_.

Example
=======

An example of programmatical usage is shown below:

.. code-block:: csharp

	var options = new GeneratorOptions { BaseOutputDirectory = @"C:\my\output" }; // create the options object
	var generator = new Generator { Options = options }; // create the generator instance
	var assembly = Assembly.GetCallingAssembly(); // get the assembly to generate files for
	generator.Generate(assembly); // generates the files