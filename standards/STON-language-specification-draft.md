Introduction
============

**Specifically Typed Object Notation** (STON) is a language-independent, human-readable data format designed for representing a wide range of structures and entities. It is largely based on *JavaScript Object Notation* (JSON), but it has a number of language features that allow representing concepts not naturally supported in JSON, such as object type information or references to elsewhere defined values.

The primary focus of STON is versatility rather than interchange, making it suitable for storing complex application-specific structures. The format still can be implemented on different platforms, and it is possible to share the same data across multiple applications. At the same time, some concepts present in STON might not have direct counterparts on certain platforms; that contrasts with JSON, whose numbers, strings, arrays and objects can be easily understood by most languages. Thus, one might decide not to use certain language features when developing an application for a specific platform, or range of platforms. With STON and JSON having different focus, it should be noted that STON does not seek to replace JSON, but rather provide tools for cases where JSON does not suffice.

The following concepts were added to STON, compared to JSON:

 - type declarations (including parameterized types, union types and collection types)
 - binary, named and code value types, aside from numbers, strings and null (JSON booleans are treated as named values)
 - references (direct, or by using relative paths)
 - object constructions (with positional or named parameters)
 - object indexed members, aside from object named members
 - extension types and members

Additionally, STON has features that, instead of introducing new concepts, make STON documents easier to read, write and maintain. Examples of such improvements include comments, string literal chains or ability to place a separator after the last element in the sequence.

*This document is the first draft, and is likely to change in the predictable future. The nomenclature and minor details might be revised, but the core concepts and rules should remain the same. Suggestions that would make this specification more concise or accessible without losing its precision are welcome.*

Table of contents
=================

 - [Introduction](#introduction)
 - [Table of contents](#table-of-contents)
 - [Scope](#scope)
 - [General terms](#general-terms)
 - [Abstract structure](#abstract-structure)
     - [Entity structure](#entity-structure)
     - [Type structure](#type-structure)
     - [Document information](#document-information)
     - [Entity semantic equivalence](#entity-semantic-equivalence)
     - [Reference addresses](#reference-addresses)
     - [Extensions](#extensions)
     - [Structure summary](#structure-summary)
 - [Syntax](#syntax)
     - [Code point groups](#code-point-groups)
     - [Tokens](#tokens)
     - [String literals](#string-literals)
     - [Spacing sequences](#spacing-sequences)
     - [Representing STON elements](#representing-ston-elements)
         - [General token sequences](#general-token-sequences)
         - [Simple value token sequences](#simple-value-token-sequences)
         - [Complex value token sequences](#complex-value-token-sequences)
         - [Type token sequences](#type-token-sequences)
         - [Reference address token sequences](#reference-address-token-sequences)
 - [Structure validity](#structure-validity)
     - [Global identifier](#global-identifier)
     - [Type](#type)
     - [Simple value](#simple-value)
     - [Complex value](#complex-value)
     - [Reference address](#reference-address)
     - [Entire document](#entire-document)
 - [Document structural equivalence](#document-structural-equivalence)
     - [Entity structural equivalence](#entity-structural-equivalence)
     - [Entity canonical form](#entity-canonical-form)
         - [Canonical sequence of elements](#canonical-sequence-of-elements)
         - [Simple value canonical form](#simple-value-canonical-form)
         - [Complex value canonical form](#complex-value-canonical-form)
         - [Type definition canonical form](#type-definition-canonical-form)
         - [Reference address canonical form](#reference-address-canonical-form)
 - [STON implementations](#ston-implementations)
     - [Regular STON implementation](#regular-ston-implementation)
     - [Strict STON implementation](#strict-ston-implementation)
     - [Limitations](#limitations)
 - [Credits](#credits)
 

Scope
=====

This specification defines:

 - the domain of elements present in STON
 - relations between specific elements
 - validity rules that specific elements must meet
 - the STON syntax (a regular and strict variation)
 - the canonical representation of STON elements
 - requirements of a valid STON implementation

The following aspects are beyond the scope of this specification:

 - representation of STON elements in programs
 - applying STON concepts to specific platforms or applications
 - specific extensions (the concept of extensions is within the scope of this specification)
 
General terms
=============

**Specifically Typed Object Notation** (STON) is the data format described in this specification. A **regular STON** term can be used, in opposition to **strict STON**.

**Strict STON** is a subset of STON designed for easier parsing than regular STON, by enforcing additional rules that allow faster determining of specific tokens purpose. Both formats can convey the same information - any valid regular STON text can be turned into a functionally equivalent strict STON text.

**Abstract STON structure** describes the concepts that are represented with STON, relations between them and rules they must follow.

A **STON entity** is the most basic element of the STON abstract structure. It is the smallest usable piece of data that can be read from STON. It can represent a sigle object.

A **STON document** is a structure built around a STON entity (a document core). The document can represent a complete object graph - the entities and relationships between them.

A **STON text** is a sequence of Unicode code points that represents a STON entity using the STON syntax. A STON document can be represented as a STON text by representing its core, but not every entity read from a valid STON text can serve as a document core.

A **strict STON text** is a STON text that represents a STON entity using the strict STON syntax.

Abstract structure
==================

This section provides a general overview of specific entities and entire documents - their structure, properties and relations between them.

Entity structure
----------------

A *STON entity* can be either a **valued entity** or **reference entity**. Each valued entity is either a **simple-valued entity** or a **complex-valued entity** (also known as **simple entity** and **complex entity**, respectively). A valued entity can represent a *definition of an object*. Each reference entity identifies a valued entity defined elsewhere in the document, and thus reference entities cannot exist on their own. A reference entity can represent a *reference to an elsewhere defined object*.

Each STON entity, valued or reference, can have a single optional **global identifier**, which can be used by reference entities. The global identifier must be unique across the entire document. If an entity has such an identifier, it is said to be **globally identified**.

Each valued STON entity can have a single optional *STON type*. STON types are described later in this specification. If a valued entity has a STON type associated, it is said to be **explicitly typed**. Otherwise, it is said to be **implicitly typed**.

---

Each simple-valued STON entity has a mandatory *simple value*, consisting of a **data type** and a **content string**.

The following data types are available: text (plain text), code, number, binary, named value, null. Thus, entities with simple values of specific data types are called text-valued, code-valued, number-valued, binary-valued, name-valued and null-valued, respectively.

Plain text and code data types represent string of characters, either arbitrary (plain text) or formatted (code). Number data type represents floating point numbers. Binary data type represents byte sequences, but can be used as an integer as well (in such case, the bits in the sequence are read binary digits of the integer, from the most to the least significant bit). Named value data type represents a fixed value of any kind, identified by its name; it can be a boolean "true"/"false" or "NaN" or an enumeration constant; the exact meaning of the named value might depend on the expected type. Null data type represents absence of a value.

The content string is a sequence of characters, empty or not. Specific data types might put requirements on the content string format. The content string must not be null for any data type other than null.

---

Each complex-valued STON entity has a mandatory *complex value*, consisting of an optional **construction**, optional **member initialization** and optional **collection initialization**. At least one of these must be present in the complex value, but more can be defined, too.

Each *construction* consists of a sequence of **positional construction parameters** (each positional parameter being a STON entity) and another sequence of **named construction parameters** (each named parameter being a pair of an arbitrary string name and associated STON entity). The construction can represent the *creation of an object* defined by the complex-valued entity.

Each *member initialization* consists of a sequence of **member bindings**, each being a pair of a binding key and a bound value (a STON entity). A member binding can be either a **named member binding** (where the binding key is an arbitrary string of characters) or an **indexed member binding** (where the binding key is a sequence of STON entities). A named member binding can be either a *regular* or *extension* named member binding (the extensions are described later in this specification). The member initialization can represent the *variable initialization of an already created object* defined by the complex-valued entity.

Each *collection initialization* contains a sequence of **collection elements**, each being a STON entity. The collection initialization can represent *adding elements to an already created collection* defined by the complex-valued entity.

---

Each reference entity has a mandatory **reference address** that unambiguously identifies a *valued entity* defined in the same STON document. Reference addresses are described later in this specification.

---

The entity data excluding the global identifier (that is, type and value for valued entities, and reference address for reference entities) is called **entity body**.

Type structure
--------------

A **STON type** can be either a **named type**, **collection type** or a **union type**.

Each named type consists of a mandatory, arbitrary string *name* and an optional sequence of *type parameters*, each being a STON type itself. The named type can be either a *regular named type* or an *extension named type* (the extensions are described later in this specification).

Each collection type contains a single *element type*, that being a STON type. It represents a type of a generic sequential collection, where each collection element is of the element type.

Each union type contains a sequence of *permitted types*, each being a STON type. It should be used in a context where more than one type is valid. It is expected to be used as a named type parameter or a collection element type, rather than on its own.

Document information
--------------------

A **STON document** is built from a valued entity, called the **document core**. It is meant to represent a complete object graph, and provides the information necessary to build the graph from scratch.

The core entity itself might not sufficiently represent an object graph, not without a context that designates it as a core entity. In particular, it might not provide enough information to resolve reference addresses or construction order, both of which are necessary to build an object graph. This is the main purpose of the STON document - to build a context around the core entity that provides complete information about the object information to be built.

Additionally, extensions can be handled or verified in the context of a STON document; a set of *known application-side extensions* might need to be provided to the document, aside from the document core. Extension are described later in the specification.

In order to resolve reference addresses, the hierarchy of contexts and global identifier mappings need to be computed. Also, the construction order cannot be determined until reference addresses are resolved.

The **hierarchy of contexts** is based on the contexts that specific entities are defined in. The following rules apply:

 - each entity has its own context, where other entities might or might not be defined
 - the context in which an entity is defined is the **parent** of that entity's context
 - the core entity is defined in a **void context**, a root context not associated with any entity
 - initialization entities (member values, index parameters and collection elements) are defined in the initialized entity's own context, and thus are that entity's **children**
 - construction parameters are defined in the same context as the constructed entity, and thus are that entity's siblings
 
Based on these rules, it is possible to determine the document's hierarchy of contexts. It is not necessary for a STON document implementation to expose the hierarchy information, but it still should be computed for internal use.

Global identifier mappings is a simple collection of identifier-entity pairs, with each identifier assigned to only one entity (identifiers must be unique). It is not necessary for a STON document implementation to expose the mappings, but they still should be computed for internal use.

With the information about the core entity, hierarchy of contexts and global identifiers mappings reference addresses can be resolved. After that, each reference entity must have assigned its value entity; if it is not possible, the whole document is invalid. Each STON document implementation must expose the information about computed referenced values.

Finally, once the references are resolved, it needs to be ensured that at least one valid object construction order exists. The construction order is a sequence of all valued entities present in the document, such that each complex entity has later position than each of its constructor parameters. It is based on a notion that an object cannot be created until all of its construction parameters are known. Also, if a reference entity is a construction parameter, its referenced value must be constructed beforehand as well - this is why construction order requires the references to be resolved first. If there is no valid construction order, the whole document is invalid as well. It is not necessary for a STON document implementation to expose any construction order, but it must be internally verified that at least one such order exists.

Entity semantic equivalence
---------------------------

In some scenarios, it is necessary to compare two STON entities with each other. This is particularly important in the context of *indexed members*, so that a sequence of STON entities can be matched against index parameters.

For that purpose, a relation of **entities semantic equivalence** is defined. It is meant to represent whether two entities correspond to the same object in a graph or not. It has the following properties:

 - the relation is reflexive, symmetric and transitive
 - each reference entity is semantically equivalent to the valued entity it references
 - a complex-valued entity cannot be semantically equivalent to any other valued entity, simple or complex
 - two valued entities whose types are not semantically equivalent cannot be equivalent themselves
 - all null-valued entities of the semantically equivalent type are semantically equivalent to each other
 - two non-null simple valued entities are semantically equivalent if and only if they have the semantically equivalent type, the same data type and the same content string
 
The relation of **types semantic equivalence** (or just **types equivalence**) has the following properties:

 - the relation is reflexive, symmetric and transitive
 - all occurrences of missing type are equivalent to each other; in other words, all implicitly typed entities have equivalent types
 - an occurrence of a missing type is never equivalent to a defined type; in other words, no implicitly typed entity can have a type equivalent to an explicitly typed entity
 - two named types are considered equivalent if they have the same name, equivalent corresponding type parameters, and they are both regular types or both extension types
 - two union types are considered equivalent if they have equivalent corresponding permitted types; the order of types is considered relevant
 - two collection types are considered equivalent if they have equivalent element types
 - two types of different kinds cannot be equivalent; i.e. named types cannot be equivalent to union types, union types to collection types, or collection types to named types

Together, both sets of rules (entity semantic equivalence and types equivalence) are sufficient to determine whether two arbitrary entities are semantically equivalent or not, in the context of a specific STON document.

A sequence of STON entities is said to **match an index** when it has the same length as the index parameters, and each entity is semantically equivalent to the corresponding index parameter. Index matching is important to consider in the context of reference addresses or validation of complex entities.
 
Reference addresses
-------------------

**Reference address** is a part of reference entity, and provides an information to identify another entity in the document tree. Each address begins with an **initial context**, identifying a specific node in the hierarchy of context, and a **relative path** composed of multiple **path segments**, that describes the way to navigate through the hierarchy to the destination.

The initial context can be:

 - the *reference defining context* (the parent context of the reference entity)
 - an *ancestor of the reference defining context* of specific order (first ancestor is the parent of the reference defining context, second ancestor is the parent of the parent etc.)
 - the *document core's own context*
 - a *globally identified entity's own context*, with a specific identifier
 
After the initial context, a sequence of path segments follows. Each path segment describes how to reach the next context from the current context. The following path segments are available:

 - an *ancestor path segment*, which changes the context to an ancestor of the current context of specific order
 - a *named member path segment*, which changes the context to a named member value of the current context, regular or extension, as defined in the member initialization
 - an *indexed member path segment*, which changes the context to an indexed member value of the current context matching a provided sequence of entities, as defined in the member initialization
 - a *collection element path segment*, which changes the context to a collection element with specific index of the current context, as defined in the collection initialization

The indexed member path segment deserves a special mention, as it has multiple special properties.

First, this is the only segment that requires a sequence of entities, and each of these must meet specific conditions. Namely, each index entity must be either a simple-valued entity (implicitly or explicitly typed) or a reference entity, and it cannot have a global identifier. All of these entities are assumed to be defined in the same context as the outer reference entity, the one the indexed member path segment belongs to. In other words, all nested entities are treated as siblings of the outer reference entity.

Second, an indexed member path segment can act like a collection element path segment under specific circumstances. This happens in two cases: when it has a single implicitly typed number-valued parameter and the current context defines no indices with single implicitly typed number-valued parameters, or when it has a single implicitly-typed binary-valued parameter and the current context defines no indices with single implicitly typed binary-valued parameters. In such case, the parameter is read as an integer to use as a collection element index, rather than used as a member index.

---

There is no way to reach an index parameter or a construction parameter through a path segment, and thus these are entirely unreachable from any context outside of their own. If either of these must be referenced elsewhere, a global identifier is required.

Whenever the context is set to a reference entity, whether as an initial context or one of subsequent contexts, it is treated as changing the context to its referenced value. If the referenced value is not yet known at the time of reaching the reference entity context, the encountered reference must be resolved immediately before any further address processing occurs.

Once the last path segment is processed, the resulting context will be the referenced value, and the address will be successfully resolved.

Extensions
----------

While STON already provides a wide range of language features that allow expressing most common-use object graphs, there still might be need for additional features, for example to provide metadata or to write documents more efficiently. For that purpose, extensions are supported in Specifically Typed Object Notation.

There are two primary mechanisms of applying an extension - extension types and extension named members.

An **extension type** can be assigned to entity that has should be handled in an unconventional manner; for example, the entity might represent not an object, but instead a command, metadata or other kind of information. It is recommended that extension types are designed in a way that their usage is consistent across all kinds of STON documents, regardless of application using these documents.

An **extension member** can be declared in the member initialization section of a complex entity, and, like entities with extension types, it should be handled in an unconventional manner, significantly different from regular entities. Some extension members might be applicable only in the context of extension types, others might be usable in regularly-typed entities as well. If the given extension member can be used in the context of some regularly-typed entities, its behavior should be consistent when applying to other regularly-typed entities as well. Also, it is recommended that extension members are designed in a way that their usage is consistent across all kinds of STON documents, regardless of application using these documents.

Also, the extensions can be handled at two points in the STON document lifetime.

**Document-side extensions** provide additional information about building the STON document, and thus are processed anytime between the parsing begins and the document is built and validated. The document must have all document-side extensions removed or replaced once completed.

In constrast, **application-side extensions** provide information used by an application when processing an already created document. The application-side extensions are the only extensions that can be found in a completed STON document. To make sure that no extensions are provided by accident, all extensions are assumed to be unknown unless stated otherwise, and if application-side extensions are present, they must be explicitly provided in a set of **known application-side extensions**. Such a set contains all valid extension member names and all valid extension type names (a valid extension type name can be invalid for an extension member, and vice versa). If any extension present in a built document is not known as application-side extension, the whole document must be treated as invalid.

Structure summary
-----------------

The following list provides a summarized overview of elements present in abstract STON structures.

 - STON entities
     - simple-valued entity
         - global identifier (optional)
         - explicit STON type (optional)
         - data type (text, code, number, binary, named value or null)
         - content string
     - complex-valued entity
         - global identifier (optional)
         - explicit STON type (optional)
         - constructor (optional)
             - positional parameters (sequence of STON entities)
             - named parameters (sequence of name-entity pairs)
         - member initialization (sequence of member bindings, optional)
             - named members (name-entity pairs, regular or extension)
             - indexed members (entity sequence-entity pairs)
         - collection (sequence of STON entities, optional)
     - reference entity
         - global identifier (optional)
         - reference address
             - initial context
                 - reference defining context
                 - ancestor of reference defining context (with specific order)
                 - document core
                 - globally identified entity (with specific identifier)
             - relative path (sequence of path segments)
                 - ancestor path segment (with specific order)
                 - named member path segment (regular or extension, with specific name)
                 - indexed member path segment (matching specific sequence of simple/reference entities)
                 - collection element path segment (with specific index)
 - STON types
     - named type (regular or extension)
         - name
         - type parameters (sequence of STON types)
     - collection type (contains a single element type)
     - union type (contains a sequence of STON types)
 - STON document
     - document core (provided, exposed)
     - known application-side extensions (optionally provided, not necessarily exposed)
     - hierarchy of contexts (computed, not necessarily exposed)
     - global identifiers mappings (computed, not necessarily exposed)
     - reference entities resolved values (computed, exposed)
     - construction order (not necessarily exposed, but its existence must be confirmed)

Syntax
======

The STON syntax is considered in the context of Unicode code point sequences, and thus STON text, too, must be a code point sequence. All code points in a STON text must be within the range from U+0001 to U+FFFF. Whenever a code point U+0000 is found when parsing a code point sequence, that code point and all subsequent code points must be ignored.

Code point groups
-----------------

Since each STON text is a sequence of Unicode code points, it is useful to identify groups of individual code points that share a common property. The following code point groups are relevant to STON syntax:

 - **letters**, which include capital letters code points (from U+0041 to U+005A) and small letters code points (from U+0061 to U+007A)
 - **decimal digits**, which include digits from 0 to 9 (code points from U+0030 to U+0039)
 - **base 2 digits**, which include 0 code point (U+0030) and 1 code point (U+0031)
 - **base 8 digits**, which include digits from 0 to 7 (code points from U+0030 to U+0037)
 - **base 16 digits**, which include *decimal digits* (U+0030 to U+0039), capital letters A to F (U+0041 to U+0046) and small letters a to f (U+0061 to U+0066)
 - **base 64 digits**, which include capital letters (U+0041 to U+005A), small letters (U+0061 to U+007A), digits (U+0030 to U+0039), minus sign (U+002D) and underscore (U+005F)
 - **whitespaces**, which include the following code points: character tabulation (U+0009), line feed (U+000A), carriage return (U+000D) and space (U+0020)
 - **newlines**, which include line feed code point (U+000A) and carriage return code point (U+000D)
 
Tokens
------

A STON text is composed of one or more STON tokens, each represented with a sequence of one or more Unicode code points. Some tokens might be represented with the same Unicode sequence, but serve different function; in such case, the purpose of the Unicode sequence can be derived from the context.

---

**CANUN identifier** (also known as *CANUN name*) is a token used where names or identifiers are needed. It begins with an underscore (U+005F) or a letter, and its subsequent code points (if any) are underscores, letters and decimal digits. The acronym stands for *Common Alpha-Numeric and Underscore Name*, called so because this family of identifiers is frequently used in programming languages.

**CANUN path separator** is a token used to form chains of CANUN identifiers. It is represented with a dot (U+002E).

**Global identifier sigil** is a token placed before a global identifier is declared. It is represented with an ampersand (U+0026).

**Glboal identifier assignment** is a token placed after a global identifier, and before the entity body. It is represented with an equality sign (U+003D).

---

**String literal** is a token beginning with a delimiter code point, containing a sequence of code points following the string literal rules, and ending with the same delimiter it started with. The string literal rules are described later in this specification. There are two kinds of string literals: **text literal**, that can be delimited with a single quote (U+0027) or a double quote (U+0022), and a **code literal**, that can be delimited with a backtick (U+0060). Text literals can represent simple text values or some names, while code literals can represent simple code values.

**String chain connector** is a token used to form chains of string literals. Two connectors are available, each with different purpose: *string appending connector*, represented with a plus sign (U+002B), and *line beginning connector*, represented with a closing angle bracket (U+003E).

**Number literal** is a token representing a simple number value. It can be composed of four parts:

 - optional sign (plus, U+002B, or minus, U+002D)
 - required integer part (sequence of one or more decimal digits)
 - optional fraction part (dot, U+002E, followed by a sequence of one or more decimal digits)
 - optional exponent part (capital letter E, U+0045, or small letter e, U+0065, optionally followed by a plus/minus sign, then followed by a sequence of one or more decimal digits)

**Binary literal** is a token representing a simple binary value. The following kinds of binary literal are available:

 - **base 2 literal**, which is composed of zero code point (U+0030), followed by capital letter B (U+0042) or small letter B (U+0062), then a sequence of one or more *base 2 digits*
 - **base 8 literal**, which is composed of zero code point, followed by capital letter O (U+004F) or small letter o (U+006F), then a sequence of one or more *base 8 digits*
 - **base 16 literal**, which is composed of zero code point, followed by capital letter X (U+0058) or small letter x (U+0078), then a sequence of one or more *base 16 digits*
 - **base 64 literal**, which is composed of zero code point, followed by capital letter Z (U+005A) or small letter z (U+007A), then a sequence of one or more *base 64 digits*, which is optionally followed by one or two equality signs (U+003D) to denote padding
 - **empty binary literal**, which is composed of zero code point, followed by capital letter N (U+004E) or small letter n (U+006E); it has no code points afterwards
 
A binary literal can be prepended with a minus sign (U+002D), right before the initial zero code point. In such case, the minus sign is a part of the literal, too, with an exception of empty binary literal.

---
 
**Construction opening token** and **construction closing token** are used to begin and end construction of a complex entity, respectively. Construction opening token is represented with an opening parenthesis (U+0028) while construction closing token is represented with a closing parenthesis (U+0029).

**Member initialization opening token** and **member initialization closing token** are used to begin and end member initialization of a complex entity, respectively. Member initialization opening token is represented with an opening curly brace (U+007B) while member initialization closing token is represented with a closing curly brace (U+007D).

**Collection initialization opening token** and **collection initialization closing token** are used to begin and end collection initialization of a complex entity, respectively. Collection initialization opening token is represented with an opening square bracket (U+005B) while collection initialization closing token is represented with a closing square bracket (U+005D).

**Sequence separator** is a token used to separate two items in a sequence. It can directly follow the last item of the sequence as well. It is represented with a comma (U+002C).

**Binding token** is used to associate a value with a parameter (such as a name or sequence of entities). It is represented with a colon (U+003A).

**Index opening token** and **index closing token** are used to begin and end a list of index parameters in member initialization, respectively. They can be used in reference addresses to access indexed members or collection elements as well. Index opening token is represented with an opening square bracket (U+005B) while index closing token is represented with a closing square bracket (U+005D).

---

**Type opening token** and **type closing token** are used to begin and end type parameters list, respectively. They can also wrap a type inside, or be used to specifically denote an implicit type. Type opening token is represented with an opening angle bracket (U+003C) while type closing token is represented with a closing angle bracket (U+003E).

**Collection type symbol** is a token used to denote collection types, right after the element type is defined. *Full collection type symbol* is represented with an opening square bracket (U+005B), followed by one or more dots (U+002E), followed by a closing square bracket (U+005D). *Short collection type symbol* is represented with the square brackets only.

**Union type separator** is a token used to separate different possible types of a given union type. It is represented with a vertical line (U+007C).

---

**Current context access token** is used to choose the initial context of a reference entity to its reference defining context. It is represented with a dollar sign (U+0024).

**Ancestor context access token** is used to access an ancestor context in a reference address. It is represented with a caret (U+005E).

**Document core access token** is used to choose the document core as the initial context of a reference entity. It is represented with a caret and asterisk sequence (U+005E, U+002A).

**Globally identified entity access token** is used to choose a globally identified entity as the initial context of a reference entity. It is represented with an at sign (U+0040).

**Member access token** is used to access a member of the current context in a reference address path. It can be used to access an ancestor context, too. It is represented with a dot (U+002E).

**Collection index access token** is used to access a collection element in a reference address path. It is represented with a number sign (U+0023).

---

**Extension token** is used to denote that a given named type is an extension type, or a given named member is an extension member. It is represented with an exclamation mark (U+0021).

String literals
---------------

Each string literal token begins with a delimiter code point, contains zero or more inner code points representing string character, and ends with the same delimiter code point it started with. This section described the structure of a valid string literal in more detail.

The inner characters cannot be control characters; only code points between U+0020 and U+FFFF are allowed. The characters between U+0000 and U+001F, as well as the original string delimiter and an escape character must be represented with an escape sequence inside the string literal. All the other characters can be provided directly inside the string literal.

Each escape sequence begins with an escape character (backslash, U+005C). Two-character and six-character escape sequences can be used. The following two-character escape sequences are available:

 - \' (U+005C, U+0027) represents a single quotation mark (U+0027)
 - \" (U+005C, U+0022) represents a double quotation mark (U+0022)
 - \\\` (U+005C, U+0060) represents a backtick (U+0060)
 - \\\\ (U+005C, U+005C) represents a backslash (U+005C)
 - \/ (U+005C, U+002F) represents a slash (U+002F)
 - \b (U+005C, U+0062) represents a backspace (U+0008)
 - \f (U+005C, U+0066) represents a form feed character (U+000C)
 - \n (U+005C, U+006D) represents a line feed character (U+000A)
 - \r (U+005C, U+0072) represents a carriage return character (U+000D)
 - \t (U+005C, U+0074) represents a horizontal tabulation character (U+0009)
 - \0 (U+005C, U+0030) represents a NUL character (U+0000)

The six character escape sequences are used to represent a code point directly, using its hexadecimal representation. It consists of two code points \u (U+005C, U+0075) and four base 16 digits code points. The represented code point can be obtained by converting the four digits to a 16-bit integer. If a character to represent does not belong to Basic Multilingual Plane (U+0000 through U+FFFF), it can be represented with a UTF-16 surrogate pair.

Overall, the sequence of characters represented by a string literal is not the sequence of Unicode code points forming the literal itself. Instead, it is obtained by ignoring the delimiters, replacing all escape sequences with the code points represented by them, and passing all the other code points directly.

Spacing sequences
-----------------

Aside from STON tokens, STON text can also contain code points sequences that can be used to improve text readability without necessarily changing the meaning of the document. These are called **spacing sequences**.

Each spacing sequence is an arbitrary combination of:

 - whitespace code points
 - single line comments, each beginning with two slash code points (U+002F), possibly containing all code points except newlines and U+0000, and ending with the first subsequent occurrence of a newline code point or the U+0000 code point, or when there are no more code points remaining
 - multiple line comments, each beginning with a slash and asterisk code point sequence (U+002F, U+002A), possibly containing all code points except U+0000, and ending with the first subsequent occurrence of an asterisk and slash code point sequence (U+002A, U+002F); the asterisk closing the comment must be different from the asterisk opening the comment, and if the comment is not closed before the first occurrence of U+0000 code point or the code point sequence ends, the whole text is syntactically invalid

A spacing sequence can appear:

 - before any STON token
 - after any STON token
 - in the middle of number literals (between any neighbouring code points of the token)
 - in the middle of binary literals (between any neighbouring code points of the token)
 
Each spacing sequence can be freely replaced with another non-empty spacing sequence without changing the meaning of the document. In most cases a spacing sequence can be removed altogether, while still resulting in functionally the same document; such a sequence is called an **insignificant spacing sequence**. On the other hand, in some cases removing the sequence would change the document or make it invalid altogether; such a sequence is called a **separating spacing sequence**. For example, a spacing sequence between two CANUN tokens is always a separating spacing sequence, because without it these tokens would merge into one. Notably, in strict STON grammar all spacing sequences are insignificant; separating spacing sequences are characteristic for regular STON.

Also, a spacing sequence cannot appear in the middle of a string literal. There, all valid whitespace code points or comment-like sequences are treated as a part of the token, and thus changing or removing them changes the meaning of the document, as opposed to insignificant or separating spacing sequences.

Representing STON elements
--------------------------

With tokens listed, it is possible to describe how specific STON token sequences can be interpreted as STON document elements. 

### General token sequences

There are frequently used general-purpose token sequences, appearing in different contexts. One of such common token sequences are *CANUN paths*. Each CANUN path begins with a CANUN identifier, after which zero or more "CANUN path separator - CANUN identifier" pairs follow. It can be used as a type name or a named value path.

Another typical token sequences are *sequences of elements*, where the exact nature of the element depends on the specific sequence (for example, sequence of entities or sequence of types). Such a sequence begins with an element, after which zero or more "sequence separator - element" pairs follow. The last element in the sequence might or might not be followed by an additional sequence separator.

---

Each *STON entity* is composed of optional global identifier token sequence, followed by a mandatory entity body token sequence. The global identifier sequence consists of a global identifier sigil, a CANUN identifier and a global identifier assigment tokens. The CANUN token is the global identifier assigned to the entity. The global identifier sigil may be omitted in regular STON, but not in strict STON.

An *entity body token sequence* depends on the kind of entity being represented. *Simple-valued entities bodies* are represented with an optional type definition sequence, followed by a mandatory simple value sequence. *Complex-valued entities bodies* are represented with an optional type definition sequence, followed by a mandatory complex value sequence. *Reference entities bodies* are represented with a reference address sequence.



### Simple value token sequences

A *simple value sequence* represents a simple value. The data type and the content depends on the specific tokens used in the token sequence. A simple value sequence can be either a text literal chain sequence, code literal chain sequence, number literal token, binary literal token or a CANUN path sequence (including "null" sequence).

A *text literal chain* represents a simple value with the text data type. It consists of a text literal, which is followed by zero or more "string chain connector - text literal" pairs. The initial text literal might be prepended with a line beginning connector without changing the meaning of the whole chain. The value content is built by concatenating text literals values in the chain. When string appending connector is used, the following text literal value is added directly to the content. When line beginning connector is used, the content is appended with a line feed character (U+000A), after which the subsequence chain literal follows.

A *code literal chain* represents a simple value with the code data type. It has a structure analogical to the text literal chain, except code literals are used in place of the text literals. The value content is built from code literals and connectors using the same rules.

It is impossible to connect text literals and code literals in the same chain in a valid STON token sequence. If such a chain appears, the whole token sequence is syntactically invalid.

---

A *number literal* represents a simple value with the number data type. A content string is obtained from the number literal by converting it to a specific notation, ensuring that all equivalent number literals are turned into equivalent STON entities. When the number is equal to zero, the content is a single zero character (U+0030). Otherwise, the content string consists of a minus sign if the number is negative, followed by one or more significant digits (with no leading or trailing zeros), followed by small letter e (U+0065), followed by a minus sign if the number is not an integer, followed by the exponent digits.

A *binary literal* represents a simple value with the binary data type. A content string is obtained from the binary literal by converting its digits to a sequence of hexadecimal digits pairs (each pair corresponding to a single byte, or 8 bits). If the literal is empty, the content is represented with an empty string. If the literal contains a minus sign at the beginning, a minus sign is added at the beginning of content as well. 

Each base 2 digit represents 1 bit, each base 8 digits represents 3 bits, each base 16 digit represents 4 bits, and each base 64 digit except for the last represents 6 bits. The last base 64 digit of the literal represents all its 6 bits if not followed by a padding characters, first 4 bits if it is followed by one padding character, or first 2 bits if it is followed by two padding characters. If the total number of bits is not a multiple of 8, zero-valued bits are added at the beginning until the nearest multiple of 8 is reached.

---

A *CANUN path* other than "null" sequence (U+006E, U+0075, U+006C, U+006C) represents a simple value with the named value data type. The CANUN path is used directly as the content string.

A CANUN path equivalent to "null" sequence (U+006E, U+0075, U+006C, U+006C) represents a simple value with the null data type. The content string must be either an empty string or null. Two null-valued entities of the same type are considered equivalent regardless of the content strings used.



### Complex value token sequences

A *complex value sequence* represents a complex value. It can be either a construction sequence, an initialization sequence or a construction sequence followed by an initialization sequence. The initialization sequence can be either a member initialization sequence, a collection initialization sequence or both of these sequences provided in any order.

---

A *construction sequence* represents a construction of a complex entity with specific parameters. If the construction is parameterless, it is represented with a construction opening token, directly followed by a construction closing token; otherwise, construction parameters must be included between the closing and opening tokens. The construction parameters can be represented with a sequence of positional parameters, or a sequence of named parameters, or a sequence of positional parameters sequence followed by a sequence of named parameters, with exactly one sequence separator between the last positional parameter and the first named parameter.

Each positional construction parameter is provided as an entity token sequence (representing the parameter value), optionally prepended with a binding token (in strict STON, the binding token is mandatory). Each named construction parameter is provided as a CANUN identifier or a text literal (representing the name), followed by a binding token, followed by an entity token sequence (representing the parameter value).

---

A *member initialization sequence* represents a member initialization of a complex entity. If the member initialization is empty, it is represented with a member initialization opening token, directly followed by a member initialization closing token; otherwise, a sequence of member bindings must be included between the closing and opening tokens.

Each named member binding is provided as a CANUN identifier or a text literal (representing the name), followed by a binding token, followed by an entity token sequence (representing the member value). The name can be optionally prefixed with an extension token, indicating the member is an extension member rather than a regular member. Each indexed member binding consists of an index opening token, followed by a sequence of entities (representing a sequence of index parameters), followed by an index closing token, followed by a binding token, followed by an entity token sequence (representing the member value).

---

A *collection initialization sequence* represents a collection initialization of a complex entity. If the collection initialization is empty, it is represented with a collection initialization opening token, directly followed by a collection initialization closing token. Otherwise, a sequence of entities must be included between the closing and opening tokens (representing the sequence of collection elements).



### Type token sequences

One of useful elements appearing in the context of type declarations is *type wrapping*. Each type wrapping begins with a type opening token, optionally followed by an inner token sequence, eventually followed by a matching type closing token. The inner token sequence of a type wrapping might contain own type wrappings, but it must not contain unmatched type opening and type closing tokens. Such a behavior of type wrappings is analogical to parentheses pairs in arithmetic expressions. If the type wrapping contains no inner sequence (it is just a type opening token directly followed by type closing token), it is said to be an *empty type wrapping*.

A *type definition sequence* represents an explicit or implicit STON type. An entity is implicitly typed when its type definition consists only of an empty type wrapping, or is omitted altogether. Explicit STON type definition can be fully enclosed in a type wrapping, making it a *wrapped type definition sequence*. Some tokens of the type definition might be outside the type wrapping, or there might be no type wrapping at all, making it a *bare type definition sequence*. In strict STON, bare type definitions are not allowed.

Any type appearing in the type definition sequence might be wrapped in a type wrapping. In such case, a type wrapping with a type as its inner token sequence is equivalent to the inner type. It is technically possible to wrap the type in arbitrarily many layers of type wrapping, without changing the meaning of the whole token sequence.

---

A *named type sequence* represents a named STON type. Its name can be provided either as a CANUN path or a text literal token (when the named type is defined outside a type wrapping, text literal names are not allowed). The name can be prefixed with an extension token, indicating the named type is an extension type rather than a regular type. When there are no type parameters, the name might be followed by an empty type wrapping sequence or end right after the name. When the type has type parameters, its name must be followed by a type wrapping with a sequence of types inside.

A *collection type sequence* represents a collection STON type. It consists of a token sequence representing the element type, directly followed by a collection type symbol, full or short (when the collection type is defined outside a type wrapping, short collection type symbol cannot be used, as it could be mistaken for an empty collection). To declare the collection of union type elements, the whole union type token sequence must be wrapped in a type wrapping; otherwise, the collection type will be created from the last element of the union type.

A *union type sequence* represents a union STON type. It begins with a token sequence representing one of permitted types, directly followed by one or more "union type separator - permitted type" pairs. If, for some reason, a union type must be used as permitted type of another union type, the inner union type must be wrapped in type wrappings.



### Reference address token sequences

A *reference address sequence* represents a reference address. It always begins with an initial context sequence, after which zero or more path segment sequences follow, one directly after another.

An *initial context sequence* represents the initial context of a reference address. It can be:

 - a single current context access token (representing the reference defining entity)
 - one or more ancestor context access tokens (representing the ancestor of the reference defining entity, of order as large as the number of tokens)
 - a single document core access token (representing the document core)
 - a single globally identified entity access token, followed by a CANUN identifier (representing the globally identified entity with the provided identifier)
 
---
 
A *path segment sequence* represents a segment of reference address' relative path.

Ancestor path segment is represented with a member access token followed by as many ancestor context access tokens as the order of ancestor being accessed.

Named member path segment is represented with a member access token, followed by the member name, represented either as a CANUN identifier or a text literal. When extension member needs to be accessed, an extension token must placed between the member access token and the name token.

Indexed member path segment is represented with an index opening token, followed by a sequence of index address entities, followed by an index closing token. The index address entity can be represented either with a simple value sequence, a type definition sequence followed by a simple value sequence, or a reference address sequence.

Collection element path segment is represented with an index opening token, followed by a collection element access token, followed by a number or a binary literal, followed by an index closing token. The literal inside this token sequence must represent a non-negative integer; otherwise, the entire token sequence is invalid.



Structure validity
==================

There is a number of conditions that a valid STON document and specific structure elements must meet, whether they have been parsed from a syntactically correct sequence of Unicode code points or built otherwise. These conditions are listed in this section.

Sometimes, terms "existing" and "non-existing" will be used with regard to some values. Generally, a non-existing value is analogical to null known from many languages - no content, no properties, etc.; while an existing value is not null, and can be assumed to have a specific structure. These terms are used to avoid confusion with null-valued entities, which themselves are existing.

Global identifier
-----------------

If an entity has an existing global identifier, that identifier must be a valid CANUN identifier.

An entity can be valid with a non-existing global identifier; in such case, it is not globally identified in the first place.

Type
----

The named type (regular or extension) is valid when it has an existing name and all of its parameters (if any) are valid and existing.

The collection type is valid as long as it has a valid, existing element type.

The union type is valid as long as its permitted types are all valid and existing.

An entity can be valid with a non-existing type; it would correspond to it being implicitly typed.

Simple value
------------

The validity of a simple value depends on the content string and the data type.

For *text values* and *code values*, any existing content string (including empty string) is valid.

For *number values*, the content string must be either "0", or it must be a character sequence composed of the significand and the exponent. The significand is a sequence of one or more decimal digits, without any zeros at the beginning or the end, optionally prepended with a minus sign (U+002D). The exponent is represented with a small letter e (U+0065), after which either zero follows, or a sequence of one or more digits of the exponent follow, beginning with a digit other than zero.

For *binary values*, the content string must be either empty string, or it must be a sequence of one or more hexadecimal digit pairs (without capital letters, i.e. A to F; their small counterparts are used instead). A non-empty hexadecimal digits sequence may be optionally prepended with a minus sign.

For *named values*, the content string must be a valid CANUN path, i.e. a sequence of CANUN names separated with a dot, without any spacing. Also, it must be different from "null" code point sequence (U+006E, U+0075, U+006C, U+006C).

For *null values*, the content string must be either empty or non-existing.

If, for some reason, the data type does not match any of listed data types, the simple value is invalid.

Complex value
-------------

The validity of a complex value depends on the validity of its specific parts.

*Construction* of a complex value is valid when there are no two named parameters with the same name in that construction and when no named parameter has a non-existing name. Also, if a STON implementation stores all construction parameters together, all positional parameters must be stored before the first named parameter. All parameters must be valid and existing entities. Also, a complex value can be valid with a non-existing construction.

*Member initialization* of a complex value is valid when there are no two regular named members with the same name, no two extension named members with the same name, and no two indexed members with matching index (if one or more indexed members use a reference entity as a parameter, that validity condition can be checked only after resolving all reference parameters). Furthermore, no named member can have a non-existing name, and no indexed member can have an empty parameter sequence or use an invalid or non-existing entity as one of its parameters. All members must have valid and existing values. Also, a complex value can be valid with a non-existing member initialization.

*Collection initialization* of a complex value has no additional validity conditions, aside from having valid and existing elements. Also, a complex value can be valid with a non-existing collection initialization.

When a complex value has no construction, no member initialization and no collection initialization, it is invalid. At least one of these components must exists, even if it is empty.

Reference address
-----------------

A valid reference address must be composed of a valid and existing initial context, and with all subsequent path segment (if any) being valid and existing.

A reference defining entity and a document core are valid initial contexts automatically. An ancestor initial context must have a positive integer order to be valid; any other order value cannot be used for the ancestor initial context. A globally identified entity initial context must have a valid CANUN name as an associated identifier; all other identifiers cannot be used for the globally identified entity initial context.

All ancestor member path segments must have a positive integer order. All named member path segments must have an existing associated name. Each indexed member path segments must have one or more entities as parameters, and each of these entities must be either a simple entity (explicitly or implicitly typed) or a reference entity; all of these entities must be valid and existing, and none can be globally identified. All collection element path segment must have a non-negative integer index.

Entire document
---------------

Each document must have an existing document core entity that has not been confirmed invalid (sometimes the entity cannot be confirmed valid before the document building is started; in particular, when indexed members with reference parameters are present).

The document core must be a valued entity; if it was a reference entity, no other entities would be possible to retrieve from that, and there would be no other entity it could reference.

No two entities in the document with an existing global identifier can have the same identifier.

All reference entities must be possible to resolve. A reference entity might be impossible to resolve if:

 - its initial context is the reference defining entity, and the reference is defined in the void context
 - its initial context is an ancestor of reference defining entity, and the ancestor with the given order does not exist
 - its initial context is a globally identified entity, and no entity has the given identifier
 - it uses an ancestor path segment in the context that does not have an ancestor with the given order
 - it uses a named member path segment in the context that defines no member with the given name and extension/regular status
 - it uses an indexed member path segment in the context that defines no matching member index, and the conditions for using the segment as a collection element path segment are not met
 - it uses a collection element path segment (or indexed member path segment as a collection element segment) in the context that defines no collection element with the given index
 - at any point in its initial context or subsequent path, it references itself, directly or through a loop of other reference entities

Once all references are resolved, indexed members with reference parameters might need additional checking, to make sure they don't match any other index in the same member initialization.

At least one valid construction order of entities must be confirmed to exist. In such an order, each complex entity must come later than each of its constructor parameter values (reflecting the fact that an object cannot be created until all parameters are known), whether defined directly for that entity or just referenced. Thus, a situation where an entity requires itself as a construction parameter or where two entities use each other as construction parameters is not allowed. Generally, all documents that use no reference entities as construction parameters can be safely assumed to have at least one valid creation order (unless they are invalid for other reason).

Finally, all entities in a valid STON document cannot use any extension named members or types, except for those recognized as application-side extensions. All document-side extensions must be handled and changed/removed by the time the document is fully validated, and all unknown extensions must be considered invalid altogether (otherwise, the unknown extensions will cause problems when processed by application, anyway).

If the document meets all of these conditions, it is considered valid, regardless of whether it can be correctly handled by the application using it or not.

Document structural equivalence
===============================

Two valid STON documents can be compared to each other for structural equivalence. Such documents cannot be distinguished in terms of their exposed abstract STON structure. In other words, when they are deterministically processed in terms of the abstract structure only, they can be swapped without changing the outcome, no matter which exact information is used. For that reason, entity structural equivalence is defined that is concerned with the way data is arranged, as opposed to entity semantic equivalence that is concerned with the data meaning. Then, two documents can be compared by checking their cores for structural equivalence.

Closely related to the notion of structural equivalence is canonical form. A document canonical form is a specific strict STON text built from a certain STON document that, when parsed, produces a structurally equivalent document. Canonical form can be defined not only for documents, but also for specific entities, values or types. In fact, a canonical form of a document is the canonical form of its core.

Each class of structurally equivalent documents has exactly one canonical form and each canonical form corresponds to only one class of documents. Thus, two documents can be checked for structural equivalence by comparing their canonical forms, regardless of the application or even the platform they are represented in.

Entity structural equivalence
-----------------------------

Structural equivalence of entities is different from their semantic equivalence. The semantic equivalence determines whether two entities represent the exact same object in the object graph, and it might be impossible to decide before references are resolved (and, by extension, before the document core is chosen). Also, the semantic equivalence is checked between entities from the same document. In contrast, structural equivalence determines whether two entities define the exact same hierarchy of entities, and thus can be freely swapped in a document without changing its meaning; therefore, structure equivalence can be determined without any document-specific information, such as the chosen core. Furthermore, structural equivalence is typically checked between entities from different documents.

Structural equivalence has the following properties:

 - structural equivalence is reflexive, symmetric and transitive
 - a valued entity is never structurally equivalent to a reference entity
 - a simple-valued entity is never structurally equivalent to a complex entity
 - two simple-valued entities are structurally equivalent when:
     - they have the same global identifier, or they both have no identifier
     - their types are structurally equivalent, or they both are implicitly typed
     - their data type is null, or they have identical content strings
 - two complex-valued entities are structurally equivalent when:
     - they have the same global identifier, or they both have no identifier
     - their types are structurally equivalent, or they both are implicitly typed
     - they define structurally equivalent constructions, or they both define no constructions
         - the number of positional parameters is the same
         - the corresponding positional parameters have structurally equivalent values
         - the number of named parameters is the same
         - the corresponding named parameters have same names and structurally equivalent values
     - they define structurally equivalent member initializations, or they both define no member initializations
         - the number of members is the same
         - the corresponding member values are structurally equivalent
         - the corresponding member kinds (regular named, extension named, index) are the same
         - the corresponding named members (regular or extension) have same names
         - the corresponding indexed members have the same number of parameters, and their corresponding parameters are structurally equivalent
     - they define structurally equivalent collection initializations, or they both define no collection initializations
         - the number of elements is the same
         - the corresponding elements are structurally equivalent
 - two reference entities are structurally equivalent when:
     - they have the same global identifier, or they both have no identifier
     - they have the same initial context
         - the initial context is of the same kind (reference defining context, ancestor context, document core or globally identified entity)
         - in case of ancestor initial context, the same ancestor order is used
         - in case of globally identified entity initial context, the same global identifier is used
     - they have the same number of relative path segments
     - the corresponding path segments are structurally equivalent
         - both segments are of the same kind (ancestor segment, named member segment, indexed member segment, collection element segment)
         - in case of ancestor path segments, the same ancestor order is used
         - in case of named member path segments, the same member name is used, and both segments refer to the same kind of member (regular or extension)
         - in case of indexed member path segments, they both have the same number of parameters, and their corresponding parameters are structurally equivalent
         - in case of collection element path segments, the same element index is used
         
Two types are structurally equivalent when they are semantically equivalent. This is because each type has exactly one representation in terms of STON abstract structure, and thus different structures will always represent different types.
         
If the documents cores meet the conditions for structural equivalence, the documents, too, are structurally equivalent.

Entity canonical form
---------------------

Entity canonical form is composed of its global identifier representation and body representation.

If the entity is globally identified, its canonical form begins with a global identifier sigil (ampersand, U+0026), followed by the identifier code point sequence, followed by the global identifier assignment token (equality sign, U+003D). Otherwise, only entity body is represented.

A body of an explicitly typed valued entity is represented with the type definition canonical form, followed by the canonical form of the entity value, simple or complex.

A body of an implicitly typed valued entity is represented only with the canonical form of the entity value, simple or complex.

A body of a reference entity is represented with the entity reference address canonical form.

### Canonical sequence of elements

Frequently, a sequence of specific elements (such as entities or types) needs to be represented in a canonical way. Canonical sequence representation is composed of canonical forms of individual elements, with neighbouring elements separated with a sequence separator (comma, U+002C). No sequence separator follows the last element, even though strict STON syntax permits it.

### Simple value canonical form

The canonical representation of a simple value depends on the data type and the content inside.

*Text-valued* and *code-valued* entities are represented with canonical string literals, based on the content string. Canonical text literals are also used to represent names present in the documents. In order to form a canonical string literal from a given content string or name string:

 - all backslash occurrences (U+005C) must be prepended with another backslash
 - all delimiter occurrences (double quote, i.e. U+0022, for text literals; backtick, i.e. U+0060, for code literals) must be prepended with a backslash
 - all backspace occurrences (U+0008) must be replaced with \b (U+005C, U+0062)
 - all form feed occurrenced (U+000C) must be replaced with \f (U+005C, U+0066)
 - all line feed occurrences (U+000A) must be replaced with \n (U+005C, U+006D)
 - all carriage return occurrences (U+000D) must be replaced with \r (U+005C, U+0072)
 - all horizontal tabulation occurrences (U+0009) must be replaced with \t (U+005C, U+0074)
 - all other Unicode code points that are not in U+0020 through U+007E range must be replaced with a six-character sequence, with two first characters being \u (U+005C, U+0075) and the remaining characters forming a hexadecimal representation of the 16-bit code point, with each digit being in range 0-9 (U+0030 through U+0039) or a-f (U+0061 through U+0066)
 - the entire content must be wrapped between two delimiters (double quotes for text literals, backticks for code literals)

*Number-valued* entities are represented directly with their content string, as the content string itself is a valid number literal.

*Binary-valued* entities is written as:

 - 0n (U+0030, U+006E), when it is empty
 - 0x (U+0030, U+0078) followed by the content string, if the content does not begin with a minus sign (U+002D)
 - -0x (U+002D, U+0030, U+0078) followed by the content string without the first character, if the content does begin with a minus sign

*Name-valued* entities are represented directly with their content string, as the content string itself is a valid CANUN path representing the named value.
 
*Null-valued* entities are represented with a "null" sequence (U+006E, U+0075, U+006C, U+006C). Since valid named values cannot have "null" sequence as a content string, there is no risk a named value and a null value are represented in the same way.
 
### Complex value canonical form

The canonical representation of a complex value is composed of the construction canonical form, followed by the member initialization canonical form, followed by the collection initialization canonical form. If a construction, member initialization or collection initialization is not defined in the complex value, it is omitted in the whole value's canonical form.

Each *construction* is canonically represented with a construction opening token (opening parenthesis, U+0028), followed by optional construction body, followed by a construction closing token (closing parenthesis, U+0029). If the construction is parameterless, the construction body is omitted. If the construction has only positional parameters, a sequence of positional parameters is used as the construction body. If the construction has only named parameters, a sequence of named parameters is used as the construction body. If the construction has both positional and named parameters, the construction body consists of a sequence of positional parameters, followed by a sequence separator, followed by a sequence on named parameters.

Each positional parameter in canonical form is represented with a binding token (colon, U+003A), followed by a canonical representation of the parameter entity. Each named parameter in canonical form is represented with a canonical text literal formed from the parameter name, followed by a binding token, followed by a canonical representation of the parameter entity.

Each *member initialization* is canonically represented with a member initialization opening token (opening curly brace, U+007B), followed by optional member initialization body, followed by a member initialization closing token (closing curly brace U+007B). If the member initialization is empty, the member initialization body is omitted. Otherwise, the member initialization body is a sequence of member bindings.

Each regular named member is represented with a canonical text literal formed from the member name, followed by a binding token, followed by a canonical representation of the member value. Each extension named member is represented analogically to the regular named members, except the name text literal is prefixed with an extension token (exclamation mark, U+0021). Each indexed member is represented with an index opening token (opening square bracket, U+005B), followed by a sequence of index parameter entities, followed by an index closing token (closing square bracket, U+005D), followed by a binding token and a canonical representation of the member value.

Each *collection initialization* is canonically represented with a collection initialization opening token (opening square bracket), optionally followed by a sequence of element entities, followed by a ollection initialization closing token (closing square bracket). If the collection initialization is empty, the sequence of element entities is omitted.

### Type definition canonical form

The type definition canonical form consists of the entity type canonical form, wrapped in a single type wrapping (between an opening angle bracket, U+003C, and a closing angle bracket, U+003E). The wrapping is required to conform to the strict STON syntax. The canonical form of a specific type depends on its kind (named type, collection type, union type).

A *named type* canonical form consists of a canonical text literal formed from the type name and an optional list of parameters. If no optional parameters are present, they are omitted; otherwise, they are represented with a type opening token (an opening angle bracket), followed by a sequence of parameter types, followed by a type closing token (a closing angle bracket). If the named type is an extension type, the name is additionaly prepended with an extension token (exclamation mark, U+0021).

A *collection type* canonical form consists of a canonical representation of the element type, followed by a short collection type symbol (opening and closing square bracket: U+005B, U+005D). If the element type is a union type, its representation must be additionally wrapped in a single type wrapping.

A *union type* canonical form consists of individual permitted types canonical representation, with neighouring types separated with a union type separator (vertical line, U+007C). Each permitted type that is a union type itself must have its representation additionally wrapped in a single type wrapping.

### Reference address canonical form

Reference address canonical form consists of a canonical representation of its initial context, followed by the path segments canonical forms directly concatenated.

The reference defining entity initial context is represented with the current context access token (a dollar sign, U+0024). The ancestor initial context is represeted with as many ancestor context access tokens (caret, U+005E) as the ancestor order. The document core initial context is represented with a document core access token (a caret and asterisk sequence, U+005E, U+002A). The globally identified entity initial context is represented with a globally identified entity access token (at sign, U+0040), followed by the identifier code point sequence.

Each ancestor path segment is represented with a member access token (dot, U+002E) and as many ancestor access tokens as the ancestor order.

Each named member path segment is represented with a member access token followed by a canonical text literal formed from the member name. If extension named member is accessed, an extension token (exclamation mark, U+0021) is placed before the name.

Each indexed member path segment is represented with an index opening token (opening square bracket, U+005B), followed by a sequence of index parameter entities, followed by an index closing token (closing square bracket, U+005D).

Each collection element path segment is represented with an index opening token, followed by a collection index access token (number sign, U+0023), followed by the element index represented as a canonical number literal, followed by an index closing token.

STON implementations
====================

Specifically Typed Object Notation was designed to be language-independent, in that it should be possible to implement in any general-purpose programming language (despite the fact that specific *concepts* reflected in the abstract STON structure might not have obvious counterparts in some languages, potentially making them more difficult to handle by specific applications). Thus, based on this very specification, it is possible to implement some or all elements of STON, whether for a specific application or for a publically available package. In everyday language, all of these could be called "STON implementations" in a narrower or broader sense, making the very concept of "STON implementation" too ambiguous. With that in mind, *STON implementation* is formally specified, so that developers using the implementation in their applications can reasonably expect specific features to be present, based on the term alone.

Regular STON implementation
---------------------------

Each **STON implementation** (also called **regular STON implementation**) must be a library, module, package or other platform-specific software component that provides:

 - an *abstract STON model* functionality
 - a *regular STON parser* functionality
 - a *canonical STON writer* functionality
 
An **abstract STON model** refers to a representation of STON abstract structure usable by the application. Depending on the platform, the model can be expressed in the terms of classes, structures or other components. The model must provide the tools for building arbitrary STON entities directly (as opposed to, for example, STON parsing) and retrieving any structural information of a STON entity that is part of the abstract structure described in this specification.

Furthermore, the abstract STON model must allow creating and validating a STON document, given an entity core, a document-side extensions handling logic and a set of known application-side extensions. Some document-side extensions might be provided inside the package, or there might be a customization mechanism that allows adding own document-side extensions. However, it must be possible to disable all document-side extension handling logic. An implementation that does not support any document-side extension processing can still be considered a valid STON implementation.

Regarding the complete STON documents, the abstract model must allow retrieving the document core (and, by extension, all its structural information). Also, given a complete STON document and one of its reference entities, it must allow easily retrieving the referenced value.

A **regular STON parser** provides a complete STON document, given a string of characters (sequence of Unicode code points), a document-side extensions handling logic and a set of known application-side extensions. Like with creating a STON document from entity core, the implementation might provide built-in document-side extensions or customization mechanism, but disabling all document-side extensions must be possible. Also, not supporting document-side extensions altogether is possible in valid STON implementations.

If the provided string of characters is not a valid STON text, does not represent a valid document core or is incompatible with the given extensions, the parsing outcome must clearly indicate the text is invalid (whether through returning a non-existent document, throwing an exception or using other platform-specific mechanism), and must be impossible to reach when a valid combination of characters and extensions is provided to the parser. The invalid outcome might or might not be differentiated based on the cause of invalidity.

When the text is valid, it must build regular and application-side extension elements as described in this specification and handle document-side extensions according to the provided handling logic. Two different implementations, given the same text, known application-side extensions and document-side extensions handling logic (or lack thereof), must create structurally equivalent documents (it can be verified by comparing resulting canonical forms). If this is not the case, at least one of these implementations is invalid (unless particularly large documents are involved, as described later in this specification).

A **canonical STON writer** provides a canonical Unicode code points sequence given a complete and valid STON document. With that, it is possible to share the same document across all the other STON implementations (as well as strict STON implementations). No document-side extension logic is needed, as each valid document must have all document-side extensions handled, and thus has none in its struture.

Strict STON implementation
--------------------------

Each **strict STON implementation** is a software component that provides:

 - an *abstract STON model* functionality
 - a *strict STON parser* functionality
 - a *canonical STON writer* functionality
 
A **strict STON parser** is similar to a regular STON parser, except a string of characters that is not a valid strict STON text must result in an invalid outcome (even if it is a valid regular STON text). Aside from that, all requirements of the regular STON parser apply.

Limitations
-----------

Ideally, any possible STON document should be possible to represent in a given implementation. However, there might be extreme cases whose handling are either impossible on a given platform, or would significantly increase the complexity implementation, possibly with added computational or memory costs for more common cases. Such extreme cases are most likely to occur with particularly large STON documents, which would be extremely uncommon and possibly represent an information that would be much more efficiently represented in a different format.

With that in mind, an exception can be made when considering STON implementations validity. If a platform-specific implementation, given unlimited memory, cannot correctly process an entity whose canonical form would exceed a length of one billion or a STON text with such a length, it still can be considered a practically valid STON implementation. All smaller entities and STON texts should be possible to process correctly, unless there are platform limitations make it impossible. A given implementation may still process the larger entities and STON texts (possibly unlimited, given enough memory), and it should still adhere to this specification in such case; however, it is not a strict requirement that any implementation must follow.

Such an upper limit has been chosen to make STON implementation conditions significantly less restrictive while still making them applicable to nearly all practical uses. For example, an unlimited, mathematically valid STON implementation would require a way to represent an arbitrarily large ancestor order in a reference address. A limited, practically valid STON implementation could instead use a 32-bit integer (signed or not), because no sufficiently small entity has more than billion levels of nesting.

Credits
=======

The STON language has been designed and described by Alice Jankowska, also known as Alphish.

Special thanks to [Mercerenies](https://github.com/Mercerenies) for providing useful feedback.