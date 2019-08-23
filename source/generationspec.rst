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

Generation spec is a class that specifies which types should be generated to TypeScript and how barrels (if any) should be generated. All generation specs must derive from *TypeGen.Core.SpecGeneration.GenerationSpec*. Inside a generation spec, it is possible to select types for generation or add barrels by invoking *AddClass*, *AddInterface*, *AddEnum* or *AddBarrel* methods in 3 different places:

* **OnBeforeGeneration method** - a virtual method invoked before files are generated. It's advised to add all C# types to generate in this method, as it exposes the currently used *GeneratorOptions* instance (available through *OnBeforeGenerationArgs*), which can be helpful in specifying additional logic based on the currently used generation options. This is not the optimal place to put *AddBarrel* calls, as the directory structure for TypeScript sources may not yet have been created in the file system.
* **OnBeforeBarrelGeneration method** - a virtual method invoked after C# types are translated and saved to TypeScript files, but before any barrels are generated. This is the place to put any *AddBarrel* method calls. Because *OnBeforeBarrelGeneration* is invoked after TypeScript types are saved in the file system, it's possible to add barrel files based on the directory structure of the generated TypeScript files (*GeneratorOptions.BaseOutputDirectory* can be used to access the "global" output directory for the generated TypeScript sources).
* **class constructor** - class constructor is invoked before files are generated, and therefore it's possible to add C# types for generation in the constructor. However, this should be avoided in favor of adding types in the *OnBeforeGeneration* method, as *OnBeforeGeneration* gives access to an instance of *GeneratorOptions* (i.e. options used for file generation), which can be further used to customize the logic of adding C# types.

It is possible to express everything that can be written with :doc:`attributes <attributes>` using generation specs, because each attribute has an equivalent in the generation spec configuration. For example, an equivalent of :code:`[TsTypeAttribute(TsType.String)]` would e.g. be :code:`AddClass<...>().Member("MemberName").Type(TsType.String)`.

To perform file generation from one or more generation specs, their class names need to be passed to the *generationSpecs* :doc:`config option <cli>`.

If any generation specs are present in the *generationSpecs* config option, TypeGen will perform file generation **only** from generation specs (i.e. not from attributes in the specified assemblies). To perform file generation from both attributes (in the specified assemblies) and generation specs, you can use the *generateFromAssemblies* config option.

Examples
========

.. code-block:: csharp

    public class MyProjectGenerationSpec : GenerationSpec
    {
        public override void OnBeforeGeneration(OnBeforeGenerationArgs args)
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
		
        public override void OnBeforeBarrelGeneration(OnBeforeBarrelGenerationArgs args)
        {
            AddBarrel(".", BarrelScope.Files); // adds one barrel file in the global TypeScript output directory containing only files from that directory
			
            AddBarrel(".", BarrelScope.Files | BarrelScope.Directories); // equivalent to AddBarrel("."); adds one barrel file in the global TypeScript output directory containing all files and directories from that directory
		
		
            // the following code for each directory creates a barrel file containing all files and directories from that directory
		
            IEnumerable<string> directories = GetAllDirectoriesRecursive(args.GeneratorOptions.BaseOutputDirectory)
                .Select(x => GetPathDiff(args.GeneratorOptions.BaseOutputDirectory, x));

            foreach (string directory in directories)
            {
                AddBarrel(directory);
            }

            AddBarrel(".");
        }
		
        private string GetPathDiff(string pathFrom, string pathTo)
        {
            var pathFromUri = new Uri("file:///" + pathFrom?.Replace('\\', '/'));
            var pathToUri = new Uri("file:///" + pathTo?.Replace('\\', '/'));

            return pathFromUri.MakeRelativeUri(pathToUri).ToString();
        }

        private IEnumerable<string> GetAllDirectoriesRecursive(string directory)
        {
            var result = new List<string>();
            string[] subdirectories = Directory.GetDirectories(directory);

            if (!subdirectories.Any()) return result;
            
            result.AddRange(subdirectories);

            foreach (string subdirectory in subdirectories)
            {
                result.AddRange(GetAllDirectoriesRecursive(subdirectory));
            }

            return result;
        }
    }
