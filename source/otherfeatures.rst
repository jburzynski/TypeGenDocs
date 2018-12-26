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

Strict null checking mode
=========================

TypeGen versions >= 1.6.0 support TypeScript strict null checking mode. To enable it, do the following:

* in CLI: set *strictNullChecks* config parameter to *true*
* programmatically: set generator options *StrictNullChecks* parameter to *true*

You can also specify how C# nullable property types will be translated to TypeScript by default, by using the *csNullableTranslation* parameter (CLI or generator options). Available choices are:

* null
* undefined
* null | undefined
* *not null and not undefined*

To override the default C# nullable types translation, you can use the following attributes on a property or field: *TsNull*, *TsNotNull*, *TsUndefined*, *TsNotUndefined* (more on these attributes above, in the *TypeGen annotations* section).

Adding files to a .NET framework project
========================================

This feature is only available in TypeGen CLI (not available in programmatical API, as the programmatical API doesn't operate on a per-project level). When this option is selected, generated TS sources will automatically be added to a .NET Framework project in the chosen project folder. This feature is only available for .NET Framework projects (not .NET Core), because .NET Core projects don't specify included files in the project file.

Creating TypeScript index file
==============================

An index file is a TypeScript file in the root TypeScript files directory, containing exports of all TypeScript types in the root directory and its subdirectories. TypeGen allows for generating an index file when generating TS files either from an assembly or a generation spec. A generated index file may look like this:

.. code-block:: typescript

    export * from './foo';
    export * from './foo-type';
    export * from './bar';
    export * from './c';
    export * from './e-class';
    export * from './f-class';
    export * from './d';

Generating default values for TS properties
===========================================

It is possible to generate default values for properties inside TS classes/interfaces, depending on the property type. An example of using this feature is described in the :doc:`CLI section <cli>`. This feature is also available in *GeneratorOptions* (programmatical API).

Custom type mappings
====================

TypeGen allows to override its default C# to TS type mappings or create new custom mappings. More details in the :doc:`CLI section <cli>`. This feature is also available in *GeneratorOptions* (programmatical API).