==================
Programmatical API
==================

If the :doc:`Command Line Interface <cli>` doesn't meet your requirements, or you just want to generate TypeScript files from the code level, you can use the TypeGen file generator class, located in TypeGen.Core assembly.

The Generator class
===================

After installing TypeGen as a `NuGet package <https://www.nuget.org/packages/TypeGen>`_, you'll have access to the *Generator* (*TypeGen.Core.Generator*) class, which is responsible for generating TypeScript files. This class is also used by :doc:`TypeGen CLI <cli>`.

Generator.Generate()
--------------------

To generate TypeScript files, the *Generate()* method is used. This method has 4 overloads:

* *Generate(GenerationSpec generationSpec)*
* *Generate(IEnumerable<Assembly> assemblies)*
* *Generate(Assembly assembly)*
* *Generate(Type type)*

All overloads except for *Generate(GenerationSpec)* generate TypeScript files based on the used attributes (from the *TypeGen.Core.TypeAnnotations* namespace). The *Generate(GenerationSpec)* takes all information from the generation spec and ignores any attributes by default. It's also possible to include the information defined in attributes when generating from a generation spec, by enabling the *useAttributesWithGenerationSpec* CLI parameter or the *GeneratorOptions.UseAttributesWithGenerationSpec* property.

Events
------

There is currently one event in the *Generator* class you can subscribe to - the *Generator.FileContentGenerated* event. This event fires whenever a TypeScript file content is generated. The event handler for this event takes an argument of type *TypeGen.Core.FileContentGeneratedArgs*, which contains the following properties (event data):

* *Type : Type* - the type for which the file was generated (can be null if the file has not been generated based on a type)
* *FilePath : string* - the generated file's path
* *FileContent : string* - the generated file content

The default handler for this event, added in the *Generator*'s constructor, saves the generated file content to the file system. There is a possibility to unsubscribe the default event handler (by using the *Generator.UnsubscribeDefaultFileContentGeneratedHandler()* method), which will cause the generated file content to not be saved in the file system. You can also re-subscribe the default handler by using the *Generator.SubscribeDefaultFileContentGeneratedHandler()* method.

Generator options
=================

The *Generator* class also has a public property named *Options*, which holds the file generation options. The options can be assigned by setting the *Generator.Options* property to an instance of *GeneratorOptions* class, or by modifying the existing *Generator.Options* instance.

The current list of options available in *GeneratorOptions* is provided in `API Reference | GeneratorOptions <http://jburzynski.net/TypeGen/api-reference/api/TypeGen.Core.GeneratorOptions.html>`_.

Example
=======

An example of programmatical usage is shown below:

.. code-block:: csharp

	var options = new GeneratorOptions { BaseOutputDirectory = @"C:\my\output" }; // create the options object
	var generator = new Generator { Options = options }; // create the generator instance
	var assembly = Assembly.GetCallingAssembly(); // get the assembly to generate files for
	generator.Generate(assembly); // generates the files