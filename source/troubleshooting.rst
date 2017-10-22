===============
Troubleshooting
===============

TypeGen.Core is not compatible with your .NET Core project
==========================================================

It's most likely a NuGet issue, which can be resolved using the following command:

.. code-block:: text

	./nuget.exe locals -clear all

No files were generated, even though the CLI exited successfully
================================================================

If you're using a .NET Core app, it's most likely that TypeGen CLI cannot resolve your project dll's dependencies. To fix it, you need to add all NuGet and other external assemblies folders to *externalAssemblyPaths* CLI parameter. Common candidate would be *C:\\Program Files\\dotnet\\sdk\\NuGetFallbackFolder*, which is used by .NET Core CLI (*dotnet* program), but is not searched by default by TypeGen.
