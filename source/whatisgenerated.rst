==================
What is generated?
==================

C# properties and fields
========================

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
	
Static, readonly and const properties/fields
--------------------------------------------

By default, any static, readonly or const C# property/field is translated as shown below:

* *static* property/field -> TypeScript *static* property (only for TypeScript classes)
* *readonly* field -> TypeScript *readonly* property (for both TypeScript classes and interfaces; additionally, the property's default value is generated for TS classes)
* *const* field -> for TypeScript classes: *static readonly* property with default value; for TypeScript interfaces: *readonly* property

To change the behavior described above, you can use a combination of *TsIgnore*, *TsDefaultValue*, *TsStatic/TsNonStatic* or *TsReadonly/TsNotReadonly* attributes on a property or field (more information in the *TypeGen attributes* section).

Default values for TypeScript properties
----------------------------------------

A default value for a TypeScript property will be generated in any of the following cases:

* The C# property/field is annotated with the *TsDefaultValue* attribute
* The C# property/field has a non-default value
* A default value is specified for the generated property's type in either the *defaultValuesForTypes* CLI parameter or the *GeneratorOptions.DefaultValuesForTypes* property

The order of checking for a default value to use is the same as listed above (first *TsDefaultValueAttribute*, then the member's value, then default value for a TS type).

Primitive property/field types
==============================

Property/field types that can be represented by TypeScript primitive types will be automatically mapped to the corresponding TypeScript types. Default mapping of C# to TypeScript files is presented below:

* *dynamic* -> *any*
* *all C# built-in numeric and byte types* -> *number*
* *string, char, Guid* -> *string*
* *bool* -> *boolean*
* *object* -> *object*
* *DateTime, DateTimeOffset* -> *Date*

Additionally, any type that implements the *IDictionary* interface (or the *IDictionary* interface itself) will be mapped to TypeScript dictionary type.
For example, *Dictionary<int, string>* will be mapped to *{ [key: number]: string; }*.

There is an option to override the default mappings or create new [C# to TypeScript] type mappings by using the *customTypeMappings* option in the CLI or *GeneratorOptions.CustomTypeMappings* in the programmatical API.

Complex property/field types
============================

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
================

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
============

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
===============

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