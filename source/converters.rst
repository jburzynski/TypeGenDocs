==========
Converters
==========

Converters allow for converting C# names to TypeScript names, by defining conversion rules between naming conventions.

A Converter is a class that defines logic for switching from one naming convention to another. There are 2 types of converters in TypeGen: *name converters* and *type name converters*. The only difference between the two is that *type name converters* can convert names depending on the C# type being generated. All *name converters* implement the *INameConverter* interface, and all *type name converters* implement *ITypeNameConverter*.

All converters available out-of-the-box in TypeGen are both *name converters* and *type name converters* (implement both interfaces). The natively available converters are:

* *PascalCaseToCamelCaseConverter* - converts PascalCase names to camelCase names
* *PascalCaseToKebabCaseConverter* - converts PascalCase names to kebab-case names
* *UnderscoreCaseToCamelCaseConverter* - converts underscore_case (or UNDERSCORE_CASE) names to camelCase names
* *UnderscoreCaseToPascalCaseConverter* - converts underscore_case (or UNDERSCORE_CASE) names to PascalCase names

Converter collections
=====================

Converter collections (or chains) are used in TypeGen to perform a conversion between naming conventions. There are 2 types of converter collections: *name converter collection* (for *name converters*) and *type name converter collection* (for *type name converters*). A converter collection defines a chain of converters that will be used to convert names.

For example, a converter collection containing two converters (in this order): **UnderscoreCaseToPascalCaseConverter** and **StripDtoConverter** (which removes the "Dto" suffix), will convert the name **MY_CLASS_DTO** in the following way:

#. First, **UnderscoreCaseToPascalCaseConverter** will convert **MY_CLASS_DTO** to **MyClassDto**
#. Then, the second converter (**StripDtoConverter**) will be used to convert **MyClassDto** to **MyClass**

Therefore, this particular converter collection will convert a name **MY_CLASS_DTO** to **MyClass**.

Writing your own converter - example
====================================

Type name and name converter
----------------------------

Let's say you want the generated TypeScript files' names to be *kebab-case*, but without the trailing *DTO*, which is present in your C# class names (let's say you have e.g. a C# class called *ProductDTO*).

One way to do this is to combine the natively available *PascalCaseToKebabCase* converter with some custom converter that strips the *DTO* suffix from the C# class name (if the class name ends with *DTO*). Such converter could be implemented as both *name* converter and *type name* converter, since it doesn't depend on the generated type (it could be reused e.g. for property names conversion).

Here's one implementation of such converter:

.. code-block:: csharp

	public class StripDtoConverter : INameConverter, ITypeNameConverter
	{
	    public string Convert(string name)
	    {
	        return ConvertName(name);
	    }
	    
	    public string Convert(string name, Type type)
	    {
	        return ConvertName(name);
	    }
	    
	    public string ConvertName(string name)
	    {
	        return name.ToUpperInvariant().EndsWith("DTO") ? name.Substring(0, name.Length - 3) : name;
	    }
	}

Now you can create a converter chain that first strips the *DTO* from the C# class name, and later converts the DTO-less name to kebab-case, by adding the following configuration to your *tgconfig.json*:

.. code-block:: json

    {
        "fileNameConverters": ["StripDtoConverter", "PascalCaseToKebabCaseConverter"]
    }


Type name converter
-------------------

Sometimes you may want to make TypeScript file (or type) names dependent on the actual C# type being generated. For example, you may want the enum file names to end with *.enum.ts* or the interface file names to end with *.interface.ts*.

Let's say you're generating all C# classes as TypeScript interfaces (by annotating them with the *ExportTsInterface* attribute). In this case, to achieve the abovementioned conversion, you may write the following type name converter:

.. code-block:: csharp

	public class TypeSuffixConverter: ITypeNameConverter
	{
	    public string Convert(string name, Type type)
	    {
	        if (type.IsClass)
	        {
	            return name + ".interface";
	        }
	        
	        if (type.IsEnum)
	        {
	            return name + ".enum";
	        }
	        
	        return name;
	    }
	}

You could then combine it with the *PascalCaseToKebabCase* converter by adding this section to your *tgconfig.json*:

.. code-block:: json

    {
        "fileNameConverters": ["PascalCaseToKebabCaseConverter", "TypeSuffixConverter"]
    }

Using converters
================

To use converters when generating your TS sources, specify a converter chain (i.e. a chain of converters identified by their class names) either in *tgconfig.json* (for :doc:`TypeGen CLI <cli>`) or in the generator options (for :doc:`programmatical usage <programmaticalapi>`).