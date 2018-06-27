=====================
TypeGen .NET CLI tool
=====================

TypeGen .NET Command Line tool can be obtained by downloading the `NuGet package <https://www.nuget.org/packages/TypeGen.DotNetCli>`_. Syntax and basic help information is presented below:

.. code-block:: text

	dotnet typegen [options] [command]
	
	Options:
	-h|--help               Show help information
	-v|--verbose            Show verbose output
	-p|--project-folder     Set project folder path(s)
	-c|--config-path        Set config path(s) to use
	
	Commands:
	generate     Generate TypeScript files
	getcwd       Get current working directory

Functionality of TypeGen .NET Command Line tool is identical to :doc:`TypeGen CLI <cli>` with two differences:

* syntax difference

* project folder(s) parameter can be omitted, in which case current directory (i.e. current project) will be used for file generation
