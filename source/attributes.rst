==================
TypeGen attributes
==================

ExportTs... attributes
======================

In order to indicate that a given type is to be exported to TypeScript, it needs to be annotated with a *ExportTs...* attribute. The available attributes are:

* *ExportTsClassAttribute*
* *ExportTsInterfaceAttribute*
* *ExportTsEnumAttribute*

Each of the attributes has one property named *OutputDir*, which can be set to indicate the output directory of the generated TypeScript file. The output directory (and all other defined paths) are resolved relatively to the generator's base output directory (*ProjectFolder* parameter in CLI).

For example, to export a class as a TypeScript interface to *my/sources* folder, the following code can be used:

.. code-block:: csharp

	[ExportTsInterface(OutputDir = "my/sources")]
	public class MyClass
	{
	    public int MyProperty { get; set; }
	}

In this case, *MyClass* (by default) will be exported to *my/sources/my-class.ts* upon generation.

TsCustomBaseAttribute
=====================

The TsCustomBaseAttribute allows for specifying a custom declaration of the base type. Can be used on classes and interfaces. If no base class is specified, base class declaration will be removed. When TsCustomBaseAttribute is used on a class/interface, its base type will **still be generated**. To avoid base type generation, use TsIgnoreBaseAttribute placed before TsCustomBaseAttribute.

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
	
If the used base type requires an *import* statement to be present, the import path can be specified as a second constructor argument. Additionally, if the type is used as an alias, the original type name can be specified as the third constructor argument.

Example:

.. code-block:: csharp

	[ExportTsClass(OutputDir = "my/sources")]
	[TsCustomBase("CustomBase", "../some/path/custom-base")]
	public class MyClass
	{
	    // ...
	}
	
This will result in the following being generated:

.. code-block:: typescript

	import { CustomBase } from "../some/path/custom-base";
	
	export class MyClass extends CustomBase {
	    // ...
	}

TsDefaultExportAttribute
========================

Used to specify if the generated TypeScript type should use a default export. There is also a possibility to enable default exports globally in the CLI (*useDefaultExport* parameter) or in the generator options (*GeneratorOptions.UseDefaultExport*).

Opt-in:

.. code-block:: csharp

	[ExportTsInterface]
	[TsDefaultExport]
	public class MyInterface
	{
	    //...
	}
	
translates to:

.. code-block:: typescript

	interface MyInterface {
	    //...
	}
	export default MyInterface;

Opt-out:

.. code-block:: csharp

	[ExportTsInterface]
	[TsDefaultExport(false)]
	public class MyInterface
	{
	    //...
	}
	
translates to:

.. code-block:: typescript

	export interface MyInterface {
	    //...
	}

TsDefaultTypeOutputAttribute
============================

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

TsDefaultValueAttribute
=======================

To indicate a default value for a TypeScript property, the *TsDefaultValue* attribute can be used:

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

**Feature for versions >= 2.0.0:** If you want to specify default values to be generated for all members with a given TypeScript type, you can do so by using the *defaultValuesForTypes* option in the CLI or *GeneratorOptions.DefaultValuesForTypes* in the programmatical API.

TsIgnoreAttribute
=================

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

TsIgnoreBaseAttribute
=====================

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

TsMemberNameAttribute
=====================

The TsMemberName attribute allows to override the generated TypeScript property name.

.. code-block:: csharp

	[ExportTsClass(OutputDir = "my/sources")]
	public class MyClass
	{
	    [TsMemberName("customProperty")] // will generate as customProperty: string;
	    public string MyProperty { get; set; }
	}

TsNull, TsNotNull, TsUndefined, TsNotUndefined attributes
=========================================================

These attributes are used to indicate an opt-in/out *null* or *undefined* type union (used in TypeScript null checking mode). Negative (*not*) attributes have precedence over positive attributes.
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

If string initializers are enabled, the above opt-in example will produce the following TypeScript:

.. code-block:: typescript

	export enum MyEnum
	{
	    A = "A",
	    B = "B"
	}
	
To specify custom logic for changing (converting) a C# enum value name to an enum initializer string, you can specify enum string initializers converters in the CLI (*enumStringInitializersConverters* parameter) or in the generator options (*GeneratorOptions.EnumStringInitializersConverters*).

TsOptionalAttribute
===================

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

TsReadonly, TsNotReadonly attributes
====================================

These attributes are used to mark a property or a field as readonly / not readonly.

C# code:

.. code-block:: csharp

	[ExportTsClass]
	public class MyClass
	{
	    public readonly string ReadonlyField = "value";
		
	    [TsNotReadonly]
	    [TsDefaultValue(null)]
	    public readonly string ReadonlyOptOut;
		
	    [TsReadonly]
	    public string ReadonlyOptIn = "value";
		
	    [TsNotReadonly]
	    public const string NotReadonlyConst = "value";
	}
	
generated TypeScript:

.. code-block:: typescript

	export class MyClass {
	    readonly readonlyField: string = "value";
	    readonlyOptOut: string;
	    readonly readonlyOptIn: string = "value";
	    static notReadonlyConst: string = "value";
	}

TsStatic, TsNotStatic attributes
================================

These attributes are used to mark a property or a field as static / not static.

C# code:

.. code-block:: csharp

	[ExportTsClass]
	public class MyClass
	{
	    public static string StaticField;
		
	    [TsNotStatic]
	    public static string StaticOptOut;
	    
	    [TsStatic]
	    public string StaticOptIn;
	    
	    [TsNotStatic]
	    public const string NotStaticConst = "value";
	}
	
generated TypeScript:

.. code-block:: typescript

	export class MyClass {
	    static staticField: string;
	    staticOptOut: string;
	    static staticOptIn: string;
	    readonly notStaticConst: string = "value";
	}

TsStringInitializersAttribute
=============================

Used to specify if TypeScript string initializers should be used for an enum. There is also a possibility to enable enum string initializers globally in the CLI (*enumStringInitializers* parameter) or in the generator options (*GeneratorOptions.EnumStringInitializers*).

Opt-in:

.. code-block:: csharp

	[ExportTsEnum]
	[TsStringInitializers]
	public enum MyEnum
	{
	    A,
	    B
	}

Opt-out:

.. code-block:: csharp

	[ExportTsEnum]
	[TsStringInitializers(false)]
	public enum MyEnum
	{
	    A,
	    B
	}

TsTypeAttribute
===============

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
	
TsTypeUnionsAttribute
=====================

The *TsTypeUnions* attribute specifies additional TypeScript type unions for a member.

Example:

.. code-block:: csharp

	[ExportTsClass]
	public class MyClass
	{
	    [TsTypeUnions("null")]
	    public int IntProperty { get; set; }
		
	    [TsTypeUnions("null", "undefined")]
	    public string StringProperty { get; set; }
		
	    [TsTypeUnions("string")]
	    public DateTime DateTimeProperty { get; set; }
	}

The above will produce the following TypeScript file:

.. code-block:: typescript

	export class MyClass {
	    intProperty: number | null;
	    stringProperty: string | null | undefined;
	    dateTimeProperty: Date | string;
	}
	