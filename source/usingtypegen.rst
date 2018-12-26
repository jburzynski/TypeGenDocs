=============
Using TypeGen
=============

1. Add `TypeGen NuGet package <https://www.nuget.org/packages/TypeGen>`_ to your project
2. Annotate your C# classes with :doc:`TypeGen-specific attributes <attributes>` or create a :doc:`generation spec <generationspec>`
3. If you intend to use more complex features of TypeGen CLI, create a :doc:`tgconfig.json <cli>` file in your project folder
4. Build your project
5. Type :code:`TypeGen generate` in :doc:`the Package Manager Console <cli>` or install `TypeGen .NET CLI tool <https://www.nuget.org/packages/TypeGen.DotNetCli>`_ and type :code:`dotnet typegen generate` in the system console
6. Instead of using TypeGen CLI, you may also use the :doc:`programmatical API <programmaticalapi>` from your code to generate TypeScript files