===============
Generation spec
===============

Generation spec is one of the two ways of selecting C# types to be generates as TypeScript sources (the other one being :doc:`attributes <attributes>`).

Using a generation spec has a few advantages over attributes:

- you can generate types from external assemblies
- configuration is not tied up to a type - you can make different configurations for the same type in different generation specs
- you can create reusable configurations and combine them together

Using generation specs is typically good for bigger projects or projects with more complex file generation logic.

Overview
========

Generation spec is a class that specifies which types should be generated to TypeScript. All generation specs must derive from *TypeGen.Core.SpecGeneration.GenerationSpec*. Inside a generation spec, it is possible to select types for generation by invoking *AddClass()*, *AddInterface()* or *AddEnum()* methods in the constructor.

It is possible to express everything that can be written with :doc:`attributes <attributes>` using generation spec, because each attribute has an equivalent in the generation spec configuration. For example, an equivalent of :code:`[TsTypeAttribute(TsType.String)]` would e.g. be :code:`AddClass<...>().Type(TsType.String)`.

By default, attributes are completely ignored when generating from a generation spec. You can change that behaviour by setting *useAttributesWithGenerationSpec* CLI option (or *GeneratorOptions.UseAttributesWithGenerationSpec*) to *true*. This will cause TypeGen to also read attribute metadata when generating from a generation spec.

After creating a generation spec, it can be passed either in *tgconfig.json* (when using :doc:`CLI <cli>`) or as an argument of *Generator.Generate(GenerationSpec generationSpec)* (when using :doc:`programmatical API <programmaticalapi>`).

Examples
========

.. code-block:: csharp

    public class MyProjectGenerationSpec : GenerationSpec
    {
        public MyProjectGenerationSpec()
        {
            AddClass<ProductDto>();

            AddInterface<CarDto>("output/directory");

            AddClass<PersonDto>()
                .Member(nameof(PersonDto.Id))  // specifying member options
                .Ignore()
                .Member(x => nameof(x.Age))    // you can specify member name with lambda
                .Type(TsType.String);

            AddInterface<SettingsDto>()
                .IgnoreBase();                 // specifying type options

            AddClass(typeof(GenericDto<>));    // specifying types by Type instance

            AddEnum<ProductType>("output/dir") // specifying an enum

            // generate everything from an assembly
			
            foreach (Type type in GetType().Assembly.GetLoadableTypes())
            {
                AddClass(type);
            }
            
            // generate types by namespace
            
            IEnumerable<Type> types = GetType().Assembly.GetLoadableTypes()
                .Where(x => x.FullName.StartsWith("MyProject.Web.Dtos"));
            foreach (Type type in types)
            {
                AddClass(type);
            }
        }
    }
