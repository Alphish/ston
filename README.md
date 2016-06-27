Introduction
============

**Specifically Typed Object Notation** (STON) is a data serialization format, largely inspired by and built upon *JavaScript Object Notation* (JSON). Its main purpose is to represent complex structures in a conventional and concise manner, so that they can be stored and read by specific applications. At the same time, it is designed to be easy for humans to read, write and maintain. With STON being a superset of JSON, any valid JSON document is also a valid STON document.

Table of contents
=================

 - [Introduction](#introduction)
 - [Table of contents](#table-of-contents)
 - [Usage notes](#usage-notes)
 - [Examples](#examples)
     - [Vector graphics](#vector-graphics)
     - [Network topology](#network-topology)
     - [Tournament](#tournament)
 - [Features](#features)
     - [Simple values](#simple-values)
     - [Complex structures](#complex-structures)
     - [Type declarations](#type-declarations)
     - [Global identifiers](#global-identifiers)
     - [Entity references](#entity-references)
     - [Extensions](#extensions)
     - [Readability and maintenance](#readability-and-maintenance)
 - [Other resources](#other-resources)
 - [Credits](#credits)

Usage notes
===========

Unlike JSON, STON has been primarily designed to be powerful, flexible and expressive rather than lightweight and language-independent. Due to these different goals, seeing STON as an alternative to JSON can be misleading; comparison to XML or YAML instead might be more appropriate.

It does not mean that simplicity has been sacrificed entirely - in fact, some structures may be represented more concisely in STON with right conventions. However, building STON documents requires more advanced parsers and additional processing (mostly validation).

Also, STON can be seen as language-independent to an extent, as it should be possible to easily represent a STON document in terms of most commonly used languages. At the same time, specific STON concepts might be reflected in some languages much more easily than in the others, and it is up to the developer to find out which concepts can be effortlessly handled by the application, and which would require costly workarounds.

Examples
========

Below are presented example valid STON documents, showing the general syntax, structure and available features. The features themselves are outlined later.

Vector graphics
---------------

    group("icon") [
        polygon {
            points: [ (16,0), (28,16), (16,32), (4,16) ],
            fill: red
            },
        group("heart") [
            polygon {
                points: [ (32,44), (64,44), (48,60) ],
                fill: red
                },
            cirsect(40, 44, rad: 8, angle_begin: 0, angle_end: 180) { fill: red },
            cirsect(56, 44, rad: 8, angle_begin: 0, angle_end: 180) { fill: red },
            ],
        ]

Network topology
----------------

    // graph example found at: http://www.analytictech.com/mb021/graphtheory.htm
    // more specifically, a figure 3
    {
        title: "Valued graph",
        nodes: &N = {
            Algeria: ("Algeria"),
            Canada: ("Canada"),
            Mexico: ("Mexico"),
            "United Kingdom": ("United Kingdom"),
            USA: ("United States of America"),          // gotta shorten that
            },
        edges: [
                (@N.Algeria, @N.Canada) { value: 2 },
                (@N.Algeria, @N."United Kingdom"),      // the default value is 1, anyway
                (@N.Canada, @N.USA) { value: 60 },
                (@N.Mexico, @N.USA) { value: 3 },
                (@N."United Kingdom", @N.USA) { value: 10 },
            ]
    }

Tournament
----------

    // it seems someone really likes explicit typing...
    tournament {
        "title" : "One week of STON 2016",
        "homepage" : url "http://the-most-amazing-tournaments-in-the-world.com/ston/",
        "entrants" : player[...] [
            player("Alice") /*not me this time*/, player("Bob"), player("Caroline"), player("Dan"),
            player("Eve"), player("Frank"), player("Grace"), player("Henry"),
            ],
        "matches" : grid<player,match> {
            [^.entrants[0], ^.entrants[1]] : match { "time" : datetime "2016-06-06 10:00", "winner" : ^^.entrants[1] },
            [^.entrants[2], ^.entrants[3]] : match { "time" : datetime "2016-06-06 12:00", "winner" : ^^.entrants[2] },
            [^.entrants[4], ^.entrants[5]] : match { "time" : datetime "2016-06-06 14:00", "winner" : ^^.entrants[4] },
            [^.entrants[6], ^.entrants[7]] : match { "time" : datetime "2016-06-06 16:00", "winner" : ^^.entrants[7] },

            [^.entrants[0], ^.entrants[2]] : match { "time" : datetime "2016-06-07 10:00", "winner" : ^^.entrants[2] },
            [^.entrants[1], ^.entrants[3]] : match { "time" : datetime "2016-06-07 12:00", "winner" : ^^.entrants[3] },
            [^.entrants[4], ^.entrants[6]] : match { "time" : datetime "2016-06-07 14:00", "winner" : ^^.entrants[6] },
            [^.entrants[5], ^.entrants[7]] : match { "time" : datetime "2016-06-07 16:00" },

            [^.entrants[0], ^.entrants[3]] : match { "time" : datetime "2016-06-08 10:00" },
            [^.entrants[1], ^.entrants[2]] : match { "time" : datetime "2016-06-08 12:00" },
            [^.entrants[4], ^.entrants[7]] : match { "time" : datetime "2016-06-08 14:00" },
            [^.entrants[5], ^.entrants[6]] : match { "time" : datetime "2016-06-08 16:00" },

            [^.entrants[0], ^.entrants[4]] : match { "time" : datetime "2016-06-09 10:00" },
            [^.entrants[1], ^.entrants[5]] : match { "time" : datetime "2016-06-09 12:00" },
            [^.entrants[2], ^.entrants[6]] : match { "time" : datetime "2016-06-09 14:00" },
            [^.entrants[3], ^.entrants[7]] : match { "time" : datetime "2016-06-09 16:00" },

            [^.entrants[0], ^.entrants[5]] : match { "time" : datetime "2016-06-10 10:00" },
            [^.entrants[1], ^.entrants[4]] : match { "time" : datetime "2016-06-10 12:00" },
            [^.entrants[2], ^.entrants[7]] : match { "time" : datetime "2016-06-10 14:00" },
            [^.entrants[3], ^.entrants[6]] : match { "time" : datetime "2016-06-10 16:00" },

            [^.entrants[0], ^.entrants[6]] : match { "time" : datetime "2016-06-11 10:00" },
            [^.entrants[1], ^.entrants[7]] : match { "time" : datetime "2016-06-11 12:00" },
            [^.entrants[2], ^.entrants[4]] : match { "time" : datetime "2016-06-11 14:00" },
            [^.entrants[3], ^.entrants[5]] : match { "time" : datetime "2016-06-11 16:00" },

            [^.entrants[0], ^.entrants[7]] : match { "time" : datetime "2016-06-12 10:00" },
            [^.entrants[1], ^.entrants[6]] : match { "time" : datetime "2016-06-12 12:00" },
            [^.entrants[2], ^.entrants[5]] : match { "time" : datetime "2016-06-12 14:00" },
            [^.entrants[3], ^.entrants[4]] : match { "time" : datetime "2016-06-12 16:00" }
            }
        }

Features
========

Specifically Typed Object Notation comes with numerous language features. Some of these are fairly typical, present in most data serialization formats. Others enhance the capabilities, allowing to represent complex structures more naturally. There are also syntactic features not affecting the represented document, instead aiming to increase readability and maintainability.

This is merely a general overview of the STON features, without getting into details of STON syntax or correct STON document structure. These are described in the [language specification draft](https://github.com/Alphish/ston/blob/master/standards/STON-language-specification-draft.md).

Also, when describing some of the features, a **CANUN format** term will be used. It stands for *Common Alpha-Numeric and Underscore Name format*, and it encompasses a commonly used family of identifiers (especially for variables in programming). Any character sequence made entirely of alphanumeric characters or underscores, not beginning with a digit, belongs to that family; all other sequences do not.

In terms of regular expressions, that format may be represented as: `^[_a-zA-Z][_a-zA-Z0-9]*$`

Simple-valued entities
----------------------

The most basic elements of STON documents are **simple-valued entities**, each representing a specific piece of data with associated data type.

**Text value** data type typically corresponds to a sequences of characters, and is represented with *text literals* or *text literal chains*. A text literal is similar to JSON string literals, except it can be wrapped between single quotes, not only double quotes. A text literal chain is an expansion upon text literals, mostly to allow splitting a text into sections or lines, for improved readability.

Example text literals:

    'A text literal wrapped in single quotes.'
    "A text literal wrapped in double quotes."

Example text literal chain:

    > "A text literal chain might or might not begin with '>' character,"
        + " but all subsequent literals must be connected with either '>' or '+' character."
    > "When '>' is used, the subsequent text literal begins a new line (line feed is added),"
        + " while '+' performs a simple concatenation of strings."
        
**Code value** data type is very similar to text data type. It, too, corresponds to a sequence of characters, except it is meant to contain a code (expression, algorithm, data structure) rather than an arbitrary text. It is represented with *code literals* or *code literal chains*. They are basically identical to their text counterparts, down to escape sequences, with the exception of delimiters - they are backticks (`` ` ``) rather than quotes. A code literal chain cannot contain any text literals; similarly, a text literal chain cannot contain any code literals.

Example code literal chain:

    >`int a = 1;`
    >`int b = 2;`
    >`int sum = a+b;`
    >`print("Hello, world!");`

**Number value** data type typically corresponds to an integer or a floating point number, and is represented with *number literals*. They are nearly the same as JSON number literals, except additional leading zeros are allowed in STON.

Example number literals: `101`; `-2.71`; `384 000`; `384e3`; `-12.34e-56`; `00042`

**Binary value** data type typically corresponds to an integer or a sequence of bytes, and is represented with *binary literals*. They always begin with 0, and are followed by base identifier, determining the range of digits. The following bases are available:

 - base 2, starts with `0b`, allows 2 digits 0 and 1
 - base 8, starts with `0o`, allows 8 digits from 0 to 7
 - base 16, starts with `0x`, allows all 10 digits, then 6 letters from A to F (case insensitive)
 - base 64, starts with `0z`, allows all 26 capital letters, then all 26 small letters, then all 10 digits, then minus (`-`), then underscore (`_`); also, equality sign (`=`) is used for padding
 - base 0, starts with `0n`, its only purpose is to represent empty binary strings

Example binary literals: `0b 01001000 01101001 00100001`; `0o644`; `-0x80`; `0zBase-64=`; `0n`

**Named value** data type typically corresponds to a predefined value recognized by the application (based on the path itself, as well as the context it appears in), and is represented with *named value paths*. Each named value path consists of one or more CANUN name segments separated with dots. The whole value cannot be `null`, as it's reserved for a null literal. Typically, a named value would represent a constant or an enumeration item.

Example named value paths: `true`; `min`; `pi`; `color.violet`; `flower.violet`; `that.name.works.too`

**Null** data type corresponds to a non-existent value, and is represented with a case-sensitive sequence of characters spelling `null`.

Complex-valued entities
-----------------------

Aside from simple-valued entites, complex values can be represented in STON documents, potentially consisting of multiple inner entites. There are three ways to define a complex structure data: *construction*, *member initialization* and *collection initialization*.

**Construction** represents creation of an object with a list of parameters. Some of these have their function defined based on position alone; these are called *positional parameters*. Others have an associated name that determines their purpose; these are called *named parameters*. The parameter name can be provided directly as a CANUN identifier or as a text literal. The names of parameters cannot repeat. Also, the positional parameters must always be defined before the named parameters.

Example constructions:

    ("John", "Doe", 1987)
    (first_name: "John", last_name: "Doe", birth_year: 1987)
    ("first-name": "John", "last-name": "Doe", "birth-year": 1987)
    ("admin", "really cool regular password", secure_only: true)

**Member initialization** represents changing the properties of already created object, and can be seen as an expansion of JSON objects. It is defined with a list of member bindings, for named members as well as indexed members. *Named member* binds a value to a name; the name might be provided directly as a CANUN identifier or as a string literal. *Indexed member* binds a value to a sequence of one or more arbitrary entities. The member names as well as indices cannot repeat.

When representing a dictionary with string keys in STON, one might choose to use named values (like it would be done in JSON) or instead opt for indices with a single string parameter. It is up to application developer to decide which is more appropriate, and to implement the application accordingly.

Example member initializations:

    {x: 20, y: 100}
    {"x": 20, "y": 100}
    {"some-member": "foo", "other-member": "bar"}
    {[1]: "one", [2]: "two", [10]: "ten"}
    {["one"]: 1, ["two"]: 2, ["ten"]: 10}

    {
        player: "Bob",
        [7,0]: (carrier, vertical),
        [2,4]: (battleship, horizontal),
        [7,7]: (submarine, horizontal),
        [3,9]: (cruiser, horizontal),
        [0,1]: (destroyer, vertical)
    }


**Collection initialization** represents adding elements to a specific, already created collection. It is defined with a list of elements to add. It is analogical to JSON arrays.

Example collection initializations:

    [4, 8, 15, 16, 23, 42]
    ["apple", orange]
    [["jagged", "arrays"], [], ["work"], ["too"]]

**Combining complex structure components** in a single entity is also possible. The construction must be defined before initializations, to reflect the fact that an object cannot be initialized before it is create. However, initializations themselves can be swapped without affecting the document.

Possible combinations of multiple components:

    ( "new" ){ "foo":"bar" }
    ( "new" )[ "first" ]
    { "foo":"bar" }[ "first" ]
    [ "first" ]{ "foo":"bar" }
    ( "new" ){ "foo":"bar" }[ "first" ]
    ( "new" )[ "first" ]{ "foo":"bar" }

Type definitions
----------------

In many cases, it is possible to reason about the expected structure based on context alone. This is why, when application expects an object of "car" type, it will sooner search for properties like "top\_speed" or "average\_fuel\_consumption" rather than "vitamins" or "flavor". For that reason, JSON can be easily used for data serialization in a variety of applications despite not having a standard way to carry type information. STON does not require providing entity types explicitly, either.

However, there might be scenarios where implicit, context-based typing proves to be difficult if not downright impossible. STON does recognize that it can represent a variety of objects, and that they can be categorized into specific types; which is why it is a notation for specifically typed objects. Thus, STON allows **explicit type definitions** whenever needed, for both simple-valued and complex-valued entities. These definitions are always provided right before the entity value they apply to.

A STON type definition can be built with:

 - *named types*, characterized by its non-empty name (dot-separated sequence of CANUN names or a string literal) and an optional list of parameters (that themselves are types); for example: `<int>`; `<set<string>>`; `<pair<string, int>>`; `<topology.path>`; `<"oddly named type">`
 - *collection types*, characterized by their inner element type, and denoted with a pair of square brackets, optionally with any number of dots between them; for example: `<string[...]>`; `<int[][]>`
 - *union types*, characterized by a sequence of permitted types, and potentially representing any of them; for example: `<string|string[]>`; `<int|long|float|double>`

Also, *type wrappings* are available to wrap specific types between them. In particular, a type wrapping might enclose the entire type definition, as shown in each of previous type definition examples. A type wrapping is necessary when a union type is used as an element type of a collection type, or a permitted type of another union type.

For example, `<<union|type|element>[]>` is different from `<union|type|element[]>`, and `<one | <two|three> | four>` is different from `<one | two | three | four>`

A type can be wrapped indefinitely between type wrappings, like here: `<<<<<"this type is very cold">>>>>`

It is not required for each type definition to be wrapped in a type wrapping. However, a named type outside a type wrapping cannot have a string literal name. Thus, `foo<bar>` or `foo<"bar">` are fine, but `"foo"<bar>` is not. Also, unwrapped collection types cannot be denoted with empty square brackets, so that they are not mistaken for empty collection initializations. Instead, at least one dot must appear between the brackets. Thus, `foo<bar[.]>[.]` or `foo<bar[]>[...]` or even `foo<bar[]>[...][...]` are fine, but `foo<bar[]>[]` or `foo<bar[]>[][...]` or `foo<bar[]>[...][]` are not.

Global identifiers
------------------

STON allows assigning a document-wide, **global identifier** to a given entity (thus making it a *globally identified entity*). The identifier must be a CANUN name, and can be declared only once in the entire document. A global identifier declaration comes before explicit type declaration if present, or the entity value otherwise. A globally identified entity can be easily used with later described reference entities.

Example globally identified entities: `&DEFAULT_SIZE = 100`; `&CENTER = point(0, 0)`; `connection("admin", &PASS = "really cool regular password", secure_only: true)`
The global identifiers are DEFAULT\_SIZE, CENTER and PASS, respectively (with PASS defining one of constructor parameters). An ampersand at the beginning of global identifier declaration is optional, but the equality sign following the identifier is required.

Reference entities
------------------

Many document formats are parsed to hierarchic structures, and STON is no exception. At the same time, STON is meant to represent a wide range of structures, including cyclic object graphs. To make that possible, **reference entities** are allowed.

Rather than defining a new value, simple or complex, a reference entity instead points to an existing valued entity defined elsewhere in the STON document, using a specific address. A global identifier can be declared for a reference entity, potentially giving the same value multiple aliases.

To better understand how addresses work, a general overview of STON document hierarchy is needed. Each STON document has a single entity, the **document core**, which directly or indirectly points to all the other entities. Also, each entity in the document has its **own context** as well as **defining/parent context**. Then, the following rules apply:

 - the document core is defined in the **void context**, a root context not associated with any entity
 - initialization entities (member values, index parameters and collection elements) are defined in the initialized entity's own context, and thus are that entity's **children**
 - construction entities are defined in the same context as the constructed entity, and thus are that entity's siblings

These rules allow determining so-called hierarchy of contexts, which allows navigation through the STON document.
 
---

Each reference address is defined by a single *initial context* and optional *relative path*, composed of one or more *path segments*. The **initial context** can be one of the following:

 - the *reference defining entity*, the context in which the reference entity is defined; denoted as `$`
 - an *ancestor* of the reference defining entity, represented with one or more carets corresponding to the ancestor order; the parent is written as `^`, while the third ancestor (parent of the parent of the parent) is written as `^^^`
 - the document *core*; denoted as `^*`
 - a *globally identified entity* of specific name; for example: `@CENTER` or `@PASS`
 
Once initial context is found, **path segments** of the relative path are handled, one by one, until the destination entity is reached. When a reference entity context is reached, the context immediately changes to its referenced value's own context. Following path segments are available:

 - an *ancestor path segment* changes the context to an ancestor of specific order, determined by a number of carets following a dot; the parent of the current context is written as `.^`, while the third ancestor is written as `.^^^`
 - a *named member path segment* changes the context to a named member of the current context, defined in the member initialization; for example: `.name` or `."some-member"`
 - an *indexed member path segment* changes the context to an indexed member of the current context, defined in the member initialization, with the index parameters being either simple-valued entities (implicitly or explicitly typed) or reference entities; for example: `[1]`; `["ten"]`; `[2,4]`; `[<float>34]`; `[@SOME\_INDEX]`
 - a *collection element path segment* changes the context to a collection element of a specific integer index, provided as a number literal or a binary literal; for example: `[#1]` or `[#0x10]`

There is no way to access a constructor parameter or an index parameter from outside of its own context using path segments. If one of these needs to be accessed from outside, a global identifier is necessary inside that entity's context.

An indexed member path segment can act as a collection element path segment. It happens when its sole parameter is an implicitly typed number or binary value, and member initialization defines no similarly structured indices. For example, the value of x member will be:

 - "foo", in case of `{ x: $[0] }[ "foo" ]`, `{ x: $[0x0], [0]: "bar" }[ "foo" ]` or `{ x: $[0], [<int>0]: "bar" }[ "foo" ]`
 - "bar", in case of `{ x: $[0], [0]: "bar" }[ "foo" ]` or `{ x: $[<int>0], [<int>0]: "bar" }[ "foo" ]`
 - invalid, in case of `{ x: $[0], [1]: "bar" }[ "foo" ]` or `{ x: $[<int>0], [0]: "bar" }[ "foo" ]`
 
Correct usage of reference addresses, while mostly intuitive, comes with a few caveats. They are yet to be described in more detail in other documents. So far, a [language specification draft](https://github.com/Alphish/ston/blob/master/standards/STON-language-specification-draft.md) is available that describes references in more precise terms.

Example of a valid (if somewhat academic) reference address:

    ^^.items[# 3]["index", @CHILD_OF_INDEX.^]."is_valid?"

Extensions
----------

While STON already has numerous language features, there still is a potential for additions that would make the documents easier to manage, or provide special data to the applications. For such cases, **extensions** are allowed. 

Some extensions might be handled at the time of parsing and validating a STON document; these are called *document-side extensions*, and they must be removed or replaced by the time the document is built. Other extensions instead might be processed later by the application, after the document is already built; these are called *application-side extensions*. To use application-side extensions, they must be declared in a set of known application-side extensions when the STON document is built.

There are two mechanisms of extending STON documents - *extension types* (as opposed to *regular types*) and *extension members* (as opposed to *regular members*).

An **extension type** is a special kind of named type, indicating that the given entity should be processed differently from the others (for example, it might carry document metadata, or represent a command). All extension types have exclamation mark before their names in STON representation. More specifically, `<!extension>` or `<!"extension">` both represent an extension type, but `<"!extension">` does not.

In general, extension types should be designed in such a way that their function is consistent no matter where they appear in the document, and their usage should be independent from specific application.

An **extension member** can be defined in an entity's *member initialization*, just like regular members; however, like extension type, it is handled in a significantly different manner. All extension members have exclamation mark before their names. More specifically, `{ !extension: null }` or `{ !"extension": null }` both represent entities with an extension member, but `{ "!extension": null }` does not.

Although an entity cannot define two regular members with the same name, it can define an extension member and regular member sharing a name, and those members are treated as distinct. Also, a reference address can refer to regular and extension named members alike. For example, in entity `{ foo: bar, !foo: baz, x: $.foo, y: $.!foo }`, `x` and `y` have values `bar` and `baz`, respectively.

Some extension members might be valid only in the context of specific extension types, others might apply to regularly typed entities as well. If an extension member can be applied to regular members, it should be designed in such a way that it acts similarly no matter what type of entity it appears in, regular types and extension types alike.

There might be STON extensions so general and useful that they are worth spreading. In such case, it is strongly recommended that the STON extension is carefully documented and standarized as soon as possible, so that independent extension implementations act consistently. This is particularly important for document-side extensions.

Readability and maintenance
---------------------------

Aside from STON general functionality, there are small but handy syntactic features that make STON documents easier to work with. Some of these were shown earlier, such as string literal chains.

STON allows using so-called *spacing sequences* that are omitted when processing the document itself; they can be used to space the document for better readability. Such sequences are composed of whitespace characters and comments. If the sequence can be safely removed from the document, it is said to be an *insignificant spacing*, and acts like insignificant whitespace in JSON. However, if removing the spacing would change the meaning of the document (for example, by joining type name and named value together), it is said to be a *separating spacing* instead; it can still be replaced with any non-empty spacing without changing the document. Character sequences typical to spacings might appear inside string literals (text or code), but they are actually parts of the literals themselves.

The following characters are treated as whitespaces: space, line feed, carriage return, horizontal tabulation.

There are two kinds of comments, typical to C-family programming languages. A *single line comment* begins with two slashes and ends at the nearest newline sequence, or the end of file. A *block comment* begins with a slash and asterisk sequence, and ends at the nearest subsequent asterisk and slash sequence.

Example STON document with comments:

    // a single line comment

    &alice = "Brillant" // this is the entire document

    /* since this is just a single entity
     * and this is pretty much what I made in the last month
     * I will add some block comment
     * so that it does not look too empty
     * like I slacked all that time
     * which I actually did
     */
     
As another convenience feature, STON allows *sequence-terminating commas*. It means that in any comma-separated sequence, a single comma might or might not be added after the last element. It might streamline a workflow a little, whether by removing unnecessary delays caused by minor syntax errors or making the sequences easier to manipulate (in particular, when commenting out last elements).

Other resources
===============

Currently, a [language specification draft](https://github.com/Alphish/ston/blob/master/standards/STON-language-specification-draft.md) is available. It describes the STON syntax and rules in more precise terms, so that consistent implementations can be created based on that. The specification and the nomenclature are still subject to change; there might be minor language revisions, too. However, the core concepts and rules should remain the same.

Language reference, tutorials and developers guide are planned for the nearest future. When it comes to implementations, .NET framework implementation is currently in progress, with the first version to be published in the first weeks of July.

Credits
=======

The STON language has been designed by Alice Jankowska, also known as Alphish. This language overview has also been written by her.

Special thanks to [Mercerenies](https://github.com/Mercerenies) for providing useful feedback.