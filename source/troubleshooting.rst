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

If you're using a .NET Core app, you most likely need to add your NuGet packages folder (typically **C:\\Users\\[user_name]\\.nuget\\packages**) to *externalAssemblyPaths* CLI config parameter. The reason for this behavior is that for .NET Core projects, NuGet stores all packages in one place, and therefore NuGet references are not copied to the project's *bin/* folder. This causes TypeGen to not be able to resolve these dependencies automatically, as they are in a different folder than the assembly itself. This behavior can also occur when generating files for any assembly that has external references which are not in the same folder (i.e. cannot be resolved automatically).
