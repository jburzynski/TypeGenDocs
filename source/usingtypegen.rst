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
* by using the *TsType* enum - the *TsType* enum contains values representing the built-in TypeScript types (object, boolean, string, number, Date and any)

If the used type requires an *import* statement to be present, the import path can be specified as a second constructor argument. Additionally, if the type is used as an alias, the original type name can be specified as the third constructor argument.

Example:

.. code-block:: csharp

	[ExportTsClass(OutputDir = "my/sources")]
	public class MyClass
	{
	    [TsType("CT", "../some/path/custom-type", "CustomType")]
	    public string CustomTypeProperty { get; set; }
	}
	
This will result in the following being generated:

.. code-block:: typescript

	import { CustomType as CT } from "../some/path/custom-type";
	
	export class MyClass {
	    customTypeProperty: CT;
	}

TsDefaultTypeOutputAttribute
----------------------------

Since TypeGen 1.2, there is an option to specify a default output path for a member's type, which will be used if no *ExportTs...* attribute is present for this type.
The path is relative to the project's folder (when using CLI) or generator's base directory (when generating programmatically).

.. code-block:: csharp

	[ExportTsClass(OutputDir = "my/sources")]
	public class MyClass
	{
	    [TsDefaultTypeOutput("custom/output/path")]
	    public CustomType CustomTypeProperty { get; set; }
	}

In this example, type *CustomType* will be generated in *custom/output/path* directory if no *ExportTs...* attribute is specified in *CustomType* definition.

TsMemberNameAttribute
---------------------

The TsMemberName attribute allows to override the generated TypeScript property name.

.. code-block:: csharp

	[ExportTsClass(OutputDir = "my/sources")]
	public class MyClass
	{
	    [TsMemberName("customProperty")] // will generate as customProperty: string;
	    public string MyProperty { get; set; }
	}
	
TsOptionalAttribute
-------------------

Marks an interface property as optional.

.. code-block:: csharp

	[ExportTsInterface]
	public class MyInterface
	{
	    [TsOptional]
	    public string MyProperty { get; set; }
	}
	
translates to:

.. code-block:: typescript

	export interface MyInterface {
	    myProperty?: string;
	}
	
TsCustomBaseAttribute
---------------------

The TsCustomBaseAttribute allows for specifying a custom declaration of the base type. Can be used on classes and interfaces. If no base class is specified, base class declaration will be removed.

.. code-block:: csharp

	[ExportTsClass]
	[TsCustomBase("CustomBase")]
	public class MyClass : MyBase
	{
	    public string MyProperty { get; set; }
	}
	
resulting TypeScript file:

.. code-block:: typescript

	export class MyClass extends CustomBase {
	    myProperty: string;
	}
	
TsIgnoreBaseAttribute
---------------------

TsIgnoreBaseAttribute causes the base class/interface declaration to be empty **and** the base type to not be generated (unless the base type itself is marked with an *ExportTs...* attribute).

.. code-block:: csharp

	[ExportTsClass]
	[TsIgnoreBase]
	public class MyClass : MyBase
	{
	    public string MyProperty { get; set; }
	}
	
generated TypeScript (MyBase is not generated if it doesn't have an *ExportTs...* attribute):

.. code-block:: typescript

	export class MyClass {
	    myProperty: string;
	}

TsNull, TsNotNull, TsUndefined, TsNotUndefined attributes
---------------------------------------------------------

These attributes are used in strict null checking mode to indicate an opt-in/out *null* or *undefined* type union. Negative (*not*) attributes have precedence over positive attributes.
E.g. this definition:

.. code-block:: csharp

	[ExportTsClass]
	public class MyClass
	{
	    [TsNull]
	    public string MyProperty { get; set; }
	}
	
will be translated to:

.. code-block:: typescript

	export class MyClass {
	    myProperty: string | null;
	}
	
The above will work only if strict null checking mode is enabled (in CLI or programmatically in the generator options).
	
What is generated?
==================

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

Native property/field types
------------------------------

Property/field types that can be represented by TypeScript built-in types will be automatically mapped to the corresponding TypeScript types. The mapping of C# to TypeScript files is presented below:

* *dynamic* -> *any*
* *all C# built-in numeric and byte types* -> *number*
* *string, char* -> *string*
* *bool* -> *boolean*
* *object* -> *object*
* *DateTime* -> *Date*

Additionally, any type that implements the *IDictionary* interface (or the *IDictionary* interface itself) will be mapped to TypeScript dictionary type.
For example, *Dictionary<int, string>* will be mapped to *{ [key: number]: string; }*.

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

Base classes
------------

Since TypeGen 1.2, base classes are automatically generated for both TypeScript classes and interfaces.

.. code-block:: csharp

	[ExportTsClass]
	public class MyClass : BaseClass
	{
	    public string MyProperty { get; set; }
	}
	
	public class BaseClass
	{
	    public int BaseField;
	}

For this code, two TypeScript files will be generated: one for *MyClass* and one for *BaseClass*:

.. code-block:: typescript

	export class MyClass extends BaseClass
	{
	    myProperty: string;
	}
	
	export class BaseClass
	{
	    baseField: number;
	}

Generic classes
---------------

TypeGen 1.2 introduces TypeScript files generation for custom generic classes.
Generic types/parameters are allowed in all parts of C# class definition: in class declaration, base class specification and as member types.
Additionally, generic type constraint (specified in *where*) will also be exported to TypeScript (**note**: *new()* and *class* specifiers will not be exported).

Example:

.. code-block:: csharp

	[ExportTsClass]
	public class MyClass<T, U> : BaseClass<T> where T: GenericClass<string>
	{
	    public T GenericProperty1 { get; set; }
	    public U GenericProperty2 { get; set; }
	    public GenericClass<int> GenericClassProperty { get; set; }
	}
	
	public class BaseClass<T>
	{
	    public T BaseProperty { get; set; }
	}
	
	public class GenericClass<T>
	{
	    public T GenericClassField;
	}
	
From this code, the following TypeScript sources will be generated:

.. code-block:: typescript

	export class MyClass<T extends GenericClass<string>, U> extends BaseClass<T> {
	    genericProperty1: T;
	    genericProperty2: U;
	    genericClassProperty: GenericClass<number>;
	}
	
	export class BaseClass<T> {
	    baseProperty: T;
	}
	
	export class GenericClass<T> {
	    genericClassField: T;
	}	

Additional features
===================

Preserving parts of a TypeScript file
-------------------------------------

Since TypeGen 1.3, there is a possibility of preserving parts of a TypeScript file, so that they won't be overridden when regenerating the file.
This can be achieved with *<custom-head>* and *<custom-body>* tags specified in the comments, as shown below:

.. code-block:: typescript

	//<custom-head>
	import { Foo } from "../bar";
	//</custom-head>

	export class Product {
	    netPrice: number;
	    vat: number;
	    
	    //<custom-body>
	    get price(): number {
	        return this.netPrice + this.vat;
	    }
	    //</custom-body>
	}

This will keep both the extra import and the *price* property intact upon file regeneration.

Multiple code fragments can be tagged with *<custom-head>* or *<custom-body>* as well (however, multiple blocks are merged into one upon file generation):

.. code-block:: typescript

	export class Product {
	    netPrice: number;
	    vat: number;
	    
	    //<custom-body>
	    get price(): number {
	        return this.netPrice + this.vat;
	    }
	    //</custom-body>
	    
	    //<custom-body>
	    get priceSpecified(): boolean {
	        return this.netPrice !== undefined;
	    }
	    //</custom-body>
	}

**Note:** :code:`//<custom-xyz>` (followed by a new line) is the only acceptable format of a tag. The following will not work:

* :code:`// <custom-xyz>`
* :code:`//<custom-xyz >`
* :code:`//<custom-xyz> some comments after this`

*<custom-head>* and *<custom-body>* tags are case-insensitive, so e.g. :code:`<CUSTOM-HEAD>` is also acceptable.

**Deprecation note:**

*<custom-head>* and *<custom-body>* tags are available since TypeGen 1.4.0. Prior to this version, the *<keep-ts>* tag was used for preserving parts of the type's definition. The *<keep-ts>* tag will still work in versions higher than 1.3.0 (unless decided otherwise), however its use is deprecated since version 1.4.0.

Strict null checking mode
-------------------------

TypeGen versions >= 1.6.0 support TypeScript2 strict null checking mode. To enable it, do the following:

* in CLI: set *strictNullChecks* config parameter to *true*
* programmatically: set generator options *StrictNullChecks* parameter to *true*

You can also specify how C# nullable property types will be translated to TypeScript by default, by using the *csNullableTranslation* parameter (CLI or generator options). Available choices are:

* null
* undefined
* null | undefined
* *not null and not undefined*

To override the default C# nullable types translation, you can use the following attributes on a property or field: *TsNull*, *TsNotNull*, *TsUndefined*, *TsNotUndefined* (more on these attributes above, in the *TypeGen annotations* section).

Converters
==========

Converters are a useful feature of TypeGen. They allow for converting C# names to TypeScript names, by defining conversion rules between naming conventions.

A Converter is a class that defines logic for switching from one naming convention to another. There are 2 types of converters in TypeGen: *name converters* and *type name converters*. The only difference between the two is that *type name converters* can convert names depending on the C# type being generated. All *name converters* implement the *INameConverter* interface, and all *type name converters* implement *ITypeNameConverter*.

All converters available out-of-the-box in TypeGen are both *name converters* and *type name converters* (implement both interfaces). The natively available converters are:

* *PascalCaseToCamelCaseConverter* - converts PascalCase names to camelCase names
* *PascalCaseToKebabCaseConverter* - converts PascalCase names to kebab-case names
* *UnderscoreCaseToCamelCaseConverter* - converts underscore_case (or UNDERSCORE_CASE) names to camelCase names
* *UnderscoreCaseToPascalCaseConverter* - converts underscore_case (or UNDERSCORE_CASE) names to PascalCase names

Converter collections
---------------------

Converter collections (or chains) are used in TypeGen to perform a conversion between naming conventions. There are 2 types of converter collections: *name converter collection* (for *name converters*) and *type name converter collection* (for *type name converters*). A converter collection defines a chain of converters that will be used to convert names.

For example, a converter collection containing two converters (in this order): **UnderscoreCaseToPascalCaseConverter** and **StripDtoConverter** (which removes the "Dto" suffix), will convert the name **MY_CLASS_DTO** in the following way:

#. First, **UnderscoreCaseToPascalCaseConverter** will convert **MY_CLASS_DTO** to **MyClassDto**
#. Then, the second converter (**StripDtoConverter**) will be used to convert **MyClassDto** to **MyClass**

Therefore, this particular converter collection will convert a name **MY_CLASS_DTO** to **MyClass**.

Using converters
----------------

In this section, only an overview of the converters has been presented. For details on how to use converters in CLI, please refer to the :doc:`Command Line Interface <cli>` section. For more information about using converters programmatically, please see the :doc:`Programmatical usage <programmaticalusage>` section. You can also create your own converter - this is described in the :doc:`Cookbook <cookbook>` section.