=============
Using TypeGen
=============

Regardless of whether you use TypeGen from the CLI or programmatically, there are some aspects common to both approaches. These aspects are covered in this section.

TypeGen annotations
===================

ExportTs... attributes
----------------------

In order to indicate that a given type is to be exported to TypeScript, it needs to be annotated with a *ExportTs...* attribute (there are, however, exceptions to this rule, which will be covered later in this section). The available attributes are:

* *ExportTsClassAttribute* (used on classes)
* *ExportTsInterfaceAttribute* (used on classes)
* *ExportTsEnumAttribute* (used on enums)

Each of the attributes has one property named *OutputDir*, which can be set to indicate the output directory of the generated TypeScript file. The output directory (and all other defined paths) are resolved relatively to the generator's base output directory (*ProjectFolder* parameter in CLI).

For example, to export a class as a TypeScript interface to *my/sources* folder, the following code can be used:

.. code-block:: csharp

	[ExportTsInterface(OutputDir = "my/sources")]
	public class MyClass
	{
	    public int MyProperty { get; set; }
	}

In this case, *MyClass* (by default) will be exported to *my/sources/my-class.ts* upon generation.

TsDefaultValueAttribute
-----------------------

To indicate a default value for a TypeScript property (available only for classes), the *TsDefaultValue* attribute can be used:

.. code-block:: csharp

	[ExportTsClass(OutputDir = "my/sources")]
	public class MyClass
	{
	    [TsDefaultValue("3.14")]
	    public int MyProperty { get; set; }
        
	    [TsDefaultValue("\"A string\"")]
	    public string MyStringProperty { get; set; }
	}

The parameter passed in *TsDefaultValue*'s constructor is a string, which is directly copied to the TypeScript file as a default value for a property. Therefore, default values for string properties have to be wrapped with *"* (double quote) or *'* (single quote) characters, in order to generate a valid TypeScript file.

TsIgnoreAttribute
-----------------

To ignore a property or field when generating TypeScript source, *TsIgnore* attribute can be used:

.. code-block:: csharp

	[ExportTsClass(OutputDir = "my/sources")]
	public class MyClass
	{
	    public int MyProperty { get; set; }
        
	    [TsIgnore]
	    public string IgnoredProperty { get; set; }
	}

In this case, the generated TypeScript class will only contain *MyProperty* property declaration.

TsTypeAttribute
---------------

There is a possibility to override the generated TypeScript type for a property or field. To do so, the *TsType* attribute can be used:

.. code-block:: csharp

	[ExportTsClass(OutputDir = "my/sources")]
	public class MyClass
	{
	    public int MyProperty { get; set; }
        
	    [TsType(TsType.String)]
	    public int StringProperty { get; set; }
	    
	    [TsType("CustomType")]
	    public string CustomTypeProperty { get; set; }
	}

The attribute's constructor allows for 2 methods of specifying the TypeScript type:

* explicitly - by typing the string value that will be used as a TypeScript type
* by using the *TsType* enum - the *TsType* enum contains values representing all primitive TypeScript types (object, boolean, string and number)
	
What is generated?
==================

TypeGen supports generating:

C# properties and fields
------------------------

Both properties and fields will be generated to TypeScript, unless marked with *TsIgnore* attribute. Fields are always declared before properties in the resulting TypeScript file.

The following class:

.. code-block:: csharp

	[ExportTsClass]
	public class MyClass
	{
	    public int MyProperty { get; set; }
	    public string MyField { get; set; }
	    public string MyProperty2 { get; set; }
	}

...will be generated to:

.. code-block:: typescript

	export class MyClass {
	    myField: string;
	    myProperty: number;
	    myProperty2: string;
	}

Primitive property/field types
------------------------------

All property/field types that can be represented by TypeScript primitive types will be automatically mapped to the corresponding TypeScript native types. The mapping of C# to TypeScript files is presented below:

* *int, long, float, double, decimal* -> *number*
* *string* -> *string*
* *bool* -> *boolean*
* *object* -> *object*

Complex property/field types
----------------------------

If the type of a property or field is a complex type (custom object or enum), the following strategy is used:

#. If the complex type is annotated with a *ExportTs...* attribute, its TypeScript source will be generated according to the path specified in the attribute.
#. If the complex type is **not** annotated with a *ExportTs...* attribute, its TypeScript source will be generated in the same folder as the containing C# type (which is currently being exported).

Therefore, it has to be kept in mind that if a complex type is not annotated with a *ExportTs...* attribute, it may be generated more than once (for each type that uses this complex type).

**Example #1**

C# sources:

.. code-block:: csharp

	[ExportTsClass(OutputDir = "my/folder1")]
	public class MyClass1
	{
	    public int MyProperty { get; set; }
	    public MyClass2 MyComplexProperty { get; set; }
	}

	public class MyClass2
	{
	    public int SomeProperty { get; set; }
	}

*my-class1.ts* (in *my/folder1*):

.. code-block:: typescript

	import { MyClass2 } from "./my-class2";

	export class MyClass1 {
	    myProperty: number;
	    myComplexProperty: MyClass2;
	}

*my-class2.ts* (in *my/folder1*):

.. code-block:: typescript

	export class MyClass2 {
	    someProperty: number;
	}

**Example #2**

C# sources:

.. code-block:: csharp

	[ExportTsClass(OutputDir = "my/folder1")]
	public class MyClass1
	{
	    public int MyProperty { get; set; }
	    public MyClass2 MyComplexProperty { get; set; }
	}

	[ExportTsClass(OutputDir = "my/folder2")]
	public class MyClass2
	{
	    public int SomeProperty { get; set; }
	}

*my-class1.ts* (in *my/folder1*):

.. code-block:: typescript

	import { MyClass2 } from "../folder2/my-class2";

	export class MyClass1 {
	    myProperty: number;
	    myComplexProperty: MyClass2;
	}

*my-class2.ts* (in *my/folder2*):

.. code-block:: typescript

	export class MyClass2 {
	    someProperty: number;
	}

Collection types
----------------

All collection or nested collection types will be exported by TypeGen. E.g., for a C# source looking like this:

.. code-block:: csharp

	[ExportTsClass]
	public class MyClass
	{
	    public int[] IntArray { get; set; }
	    public IEnumerable<int> IntEnumerable { get; set; }
	    public IEnumerable<int[]> IntEnumArrayCombo { get; set; }
	    public IEnumerable<IList<int[]>> IntEnumListArrayCombo { get; set; }
	}

...the following TypeScript file will be generated:

.. code-block:: typescript

	export class MyClass {
	    intArray: number[];
	    intEnumerable: number[];
	    intEnumArrayCombo: int[][];
	    intEnumListArrayCombo: int[][][];
	}

Converters
==========

Converters are a useful feature of TypeGen. They allow for converting C# names to TypeScript names, by defining conversion rules between naming conventions.

A Converter is a class that defines logic for switching from one naming convention to another. There are 2 types of converters in TypeGen: *name converters* and *type name converters*. The only difference between the two is that *type name converters* can convert names depending on the C# type being generated. All *name converters* implement the *INameConverter* interface, and all *type name converters* implement *ITypeNameConverter*.

All converters available out-of-the-box in TypeGen are both *name converters* and *type name converters* (implement both interfaces). The natively available converters are:

* *NoChangeConverter* - the default converter, has a special use in TypeGen - it's always used at the beginning of the converter chain (the converter chain will be described later). Shouldn't be used outside TypeGen.Core.
* *PascalCaseToCamelCaseConverter* - converts PascalCase names to camelCase names
* *PascalCaseToKebabCaseConverter* - converts PascalCase names to kebab-case names
* *UnderscoreCaseToCamelCaseConverter* - converts underscore_case (or UNDERSCORE_CASE) names to camelCase names
* *UnderscoreCaseToPascalCaseConverter* - converts underscore_case (or UNDERSCORE_CASE) names to PascalCase names

Converter collections
---------------------

Converter collections (or chains) are used in TypeGen to perform a conversion between naming conventions. There are 2 types of converter collections: *name converter collection* (for *name converters*) and *type name converter collection* (for *type name converters*). A converter collection defines a chain of converters that will be used to convert names.

For example, a converter collection containing two converters (in this order): *UnderscoreCaseToPascalCaseConverter* and *PascalCaseToCamelCaseConverter*, will convert the name *"MY_CLASS"* in the following way:

#. First, *UnderscoreCaseToPascalCaseConverter* will convert *MY_CLASS* to *MyClass*
#. Then, the second converter (*PascalCaseToCamelCaseConverter*) will be used to convert *MyClass* to *myClass*

Therefore, this particular converter collection will convert a name *MY_CLASS* to *myClass*.

Using converters
----------------

In this section, only an overview of the converters has been presented. For details on how to use converters in CLI, please refer to the :doc:`Command Line Interface <cli>` section. For more information about using converters programmatically, please see the :doc:`Programmatical usage <programmaticalusage>` section. You can also create your own converter - this is described in the :doc:`Cookbook <cookbook>` section.