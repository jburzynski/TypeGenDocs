==============
Other features
==============

Preserving parts of a TypeScript file
=====================================

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

Strict null checking
====================

You can specify how C# nullable property/field types will be translated to TypeScript by default, by using the *csNullableTranslation* parameter (CLI or generator options). Available choices are:

* type | null
* type | undefined
* type | null | undefined
* type (not null and not undefined)

To override the default C# nullable types translation or to specify a type union for non-nullable types, you can use the following attributes on a property or field: *TsNull*, *TsNotNull*, *TsUndefined*, *TsNotUndefined* (you can find more information on these attributes in the *TypeGen attributes* section).

Adding files to a .NET framework project
========================================

This feature is only available in TypeGen CLI (not available in programmatical API, as the programmatical API doesn't operate on a per-project level). When this option is selected, generated TS sources will automatically be added to a .NET Framework project in the chosen project folder. This feature is only available for .NET Framework projects (not .NET Core), because .NET Core projects don't specify included files in the project file.

Creating TypeScript index file
==============================

An index file is a TypeScript file in the root TypeScript files directory, containing exports of all TypeScript types in the root directory and its subdirectories. TypeGen allows for generating an index file when generating TS files either from an assembly or a generation spec. A generated index file may look like this:

.. code-block:: typescript

    export * from './foo';
    export * from './bar';
    export * from './baz';
    export * from './foo-bar';
    export * from './foo-bar-baz';

Default values for TS properties based on the type
==================================================

It is possible to generate default values for properties inside TS classes/interfaces, depending on the property type. The [TS type -> default value] mappings can be specified either in the *defaultValuesForTypes* CLI parameter or in the *GeneratorOptions.DefaultValuesForTypes* property.

Custom type mappings
====================

TypeGen allows to override its default C# to TS type mappings or create new custom mappings. The way to define mappings is via either the *customTypeMappings* CLI parameter or the *GeneratorOptions.CustomTypeMappings* property.

Default exports
===============

There is a possibility to use default exports instead of named exports when generating TypeScript types. This feature can be enabled/disabled globally, by setting the *useDefaultExport* CLI parameter or the *GeneratorOptions.UseDefaultExport* property, or a one-time opt-in/out can be used with either the `TsDefaultExportAttribute` or the `DefaultExport(bool)` generation spec method.