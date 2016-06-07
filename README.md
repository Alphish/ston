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

Also, STON can be seen as language-independent to an extent, as it should be possible to easily represent a STON document in terms of most commonly used languages. At the same time, specific STON features might be reflected in some languages much more easily than in the others, and it is up to the developer to find out which features can be effortlessly handled by the application, and which would require costly workarounds.

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

This is merely a general overview of the STON features, without getting into details of STON syntax or correct STON document structure. More detailed descriptions are yet to be provided in language reference, tutorial and formal specification.

Also, when describing some of the features, a **CANUN format** term will be used. It stands for *Common Alpha-Numeric and Underscore Name format*, and it encompasses a commonly used family of identifiers (especially for variables in programming). Any character sequence made entirely of alphanumeric characters or underscores, not beginning with a digit, belongs to that family; all other sequences do not.

In terms of regular expressions, that format may be represented as: `^[_a-zA-Z][a-zA-Z0-9]*$`

Simple values
-------------

The most basic elements of STON documents are **simple values**, each representing a specific piece of data with associated data type.

**Text value** data type typically corresponds to a sequences of characters, and is represented with *string literals* or *string literal chains*. A string literal is similar to JSON string literals, except it can be wrapped between single quotes, not only double quotes. A string literal chain is an expansion upon string literals, mostly to allow splitting a string into sections or lines, for improved readability.

Example string literals:

    'A string literal wrapped in single quotes.'
    "A string literal wrapped in double quotes."

Example string literal chain:

    > "A string literal chain might or might not begin with '>' character,"
        + " but all subsequent strings must be connected with either '>' or '+' character."
    > "When '>' is used, the subsequent string begins a new line (line feed is added),"
        + " while '+' performs a simple concatenation of strings."

**Number value** data type typically corresponds to an integer or a floating point number, and is represented with *number literals*. They are nearly the same as JSON number literals, except additional leading zeros are allowed in STON.

Example number literals: `101`; `-2.71`; `384 000`; `384e3`; `-12.34e-56`; `00042`

**Binary value** data type typically corresponds to an integer or a sequence of bytes, and is represented with *binary literals*. They always begin with 0, and are followed by base identifier, determining the range of digits. The following bases are available:

 - base 2, starts with `0b`, allows 2 digits 0 and 1
 - base 8, starts with `0o`, allows 8 digits from 0 to 7
 - base 16, starts with `0x`, allows all 10 digits, then 6 letters from A to F (case insensitive)
 - base 64, starts with `0z`, allows all 26 capital letters, then all 26 small letters, then all 10 digits, then minus (`-`), then underscore (`_`)

Example binary literals: `0b 01001000 01101001 00100001`; `0o644`; `-0x80`; `0zBase-64=`

**Named value** data type typically corresponds to a predefined value recognized by the application (based on the path itself, as well as the context it appears in), and is represented with *named value paths*. Each named value path consists of one or more CANUN name segments separated with dots. The first segment must not be a null literal. Typically, a named value would represent a constant or an enumeration item.

Example named value paths: `true`; `min`; `pi`; `color.violet`; `flower.violet`; `that.name.works.too`

**Null** data type corresponds to a missing value, and is represented with *null literals*. A null literal is a sequence of characters spelling "NULL", in any combination of lower and upper case.

Example null literals: `null`; `Null`; `nuLL`

Complex structures
------------------

Aside from simple values, more advanced structures can be represented in STON documents, potentially consisting of multiple values. There are three ways to define a complex structure data: *constructor*, *member initialization* and *collection initialization*.

**Constructor** is defined as a list of parameters. Some of these have their function defined based on position alone; they are called *positional parameters*. Others have an associated name that determines their purpose; they are called *named parameters*. The parameter names can be provided as a direct CANUN identifier or a string literal. The names of parameters cannot repeat. Also, the positional parameters must always be defined before the named parameters.

Example constructors:

    ("John", "Doe", 1987)
    (first_name: "John", last_name: "Doe", birth_year: 1987)
    ("first-name": "John", "last-name": "Doe", "birth-year": 1987)
    ("admin", "really cool regular password", secure_only: true)

**Member initialization** is defined as a list of initialization statements, for named members as well as indexed members. It can be seen as an expansion of the JSON objects. *Named member* binds a value to a name; the name might be provided as a direct CANUN identifiers or a string literal. *Indexed member* binds a value to one or more arbitrary entities. The member names as well as indices cannot repeat.

When representing a dictionary in STON, one might choose to use named values (like it would be done in JSON string) or instead opt for indices with a single string parameter. It is up to application developer to decide which is more appropriate, and to prepare the application accordingly.

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


**Collection initialization** is defined as a list of elements to populate the collection with. It is analogical to JSON arrays.

Example collection initializations:

    [4, 8, 15, 16, 23, 42]
    ["apple", orange]
    [["jagged", "arrays"], [], ["work"], ["too"]]

**Combining complex structure components** in a single entity is also possible. The constructor must be defined before initializations, but initializations themselves can be swapped without affecting the document.

Possible combinations of multiple components:

    ( "new" ){ "foo":"bar" }
    ( "new" )[ "first" ]
    { "foo":"bar" }[ "first" ]
    [ "first" ]{ "foo":"bar" }
    ( "new" ){ "foo":"bar" }[ "first" ]
    ( "new" )[ "first" ]{ "foo":"bar" }

Type declarations
-----------------

In many cases, it is possible to reason about the expected structure based on context alone. This is why, when application expects an object of "car" type, it will sooner search for properties like "top\_speed" or "average\_fuel\_consumption" rather than "vitamins" or "flavor". For that reason, JSON can be easily used for data serialization in a variety of applications despite not having a standard way to carry type information. STON does not require providing entity types explicitly, either.

However, there might be scenarios where implicit, context-based typing proves to be difficult if not downright impossible. STON does recognize that it can represent a variety of objects, and that they can be categorized into specific types; which is why it is a notation for specifically typed objects. Thus, STON allows **explicit type declarations** whenever needed, for simple values as well as complex structures. These declarations are always provided right before the entity value they apply to.

A STON type declaration can be built with:

 - a *named type*, characterized by its non-empty name (dot-separated sequence of CANUN names or a string literal) and an optional list of parameters (that themselves are types); for example: `<int>`; `<set<string>>`; `<pair<string, int>>`; `<topology.path>`; `<"oddly named type">`
 - an *array type*, characterized by its inner element type; for example: `<string[...]>`; `<int[][]>`
 - a *union type*, characterized by a list of possible types and potentially representing any of them; for example: `<string|string[]>`; `<int|long|float|double>`

These types can be freely nested and combined, like here: `<set< pair<int|long|"whatever it is"<"I have no idea">, string[]>[] >>`

The angle brackets aren't required around the type declaration. However, in such case the top-level named type cannot have a string literal name. Thus, `foo<bar>` or `foo<"bar">` are fine, but `"foo"<bar>` is not. Also, the top-level array types must not be denoted with empty square brackets; instead, at least one dot must appear between the brackets (otherwise, the array mark may be mistaken for collection initialization). Thus, `foo<bar[.]>[.]` or `foo<bar[]>[...]` or even `foo<bar[]>[...][...]` are fine, but `foo<bar[]>[]` or `foo<bar[]>[][...]` or `foo<bar[]>[...][]` are not.

Global identifiers
------------------

STON allows assigning a document-wide, **global identifier** to a given entity (thus making it a *globally identified entity*). The identifier must be a CANUN name, and can be declared only once in the entire document. A global identifier declaration comes before explicit type declaration if present, or the entity value otherwise. A globally identified entity can be easily used with later described reference entities.

Example globally identified entities: `&DEFAULT_SIZE = 100`; `&CENTER = point(0, 0)`; `connection("admin", &PASS = "really cool regular password", secure_only: true)`
The global identifiers are DEFAULT\_SIZE, CENTER and PASS, respectively (with PASS defining one of constructor parameters). An ampersand at the beginning of global identifier declaration is optional, but the equality sign following the identifier is required.

Entity references
-----------------

Many document formats are parsed to hierarchic structures, and STON is no exception. At the same time, many applications operate on structures that resemble arbitrary graphs rather than object trees. To make such a structure possible to represent in somewhat hierarchic STON, **entity references** are allowed. Rather than defining a new value, a reference instead points to an existing entity defined elsewhere in the STON document, using a specific address. For that reason, an explicit type declaration is forbidden on a reference (as it can mismatch the referenced entity's type). At the same time, a global identifier can still be declared on a reference, potentially giving the same entity multiple aliases.

To better understand how addresses work, a general overview of STON document hierarchy might be necessary. Each STON document has exactly one top-level entity, the *root* of the document tree. Then, a simple entity can be seen as a tree *leaf*, while a complex entity can be seen as tree *branch*, with each constructor parameter, member value, index parameter and collection element being its *child*. In that model, an entity reference might be accessed as a child of a complex entity, while acting as the referenced entity once reached. Naturally, a complex entity is a *parent* of each of its children; also described as the *first ancestor* or first order ancestor. Then, the ancestor of the order *N+1* is the parent of the *N-th ancestor*. Having these terms defined, it is now possible to explain reference addresses.

In general, when navigating through a document tree, one can start from a certain entity (called *initial context*) and from there go through a chain of related entities (changing the context) until reaching the destination. The **initial context** can be one of the following:

 - the *reference defining entity*, i.e. the complex entity that uses a reference as a parameter/value/element; denoted as `$`
 - the *N-th ancestor* of the reference defining entity, represented with one or more carets corresponding to the ancestor order; a parent is written as `^`, while a third ancestor is written as `^^^`
 - the document *root*; denoted as `^*`
 - a *globally identified entity* of specific name; for example: `@CENTER` or `@PASS`
 
Then, **path segments** might be appended to the initial context to specify the actual destination. When the segment points to an entity reference, it switches to the referenced entity altogether. Following path segments are available:

 - an *ancestor path segment* changes the context to an ancestor of specific order, determined by a number of carets following a dot; the parent of the current context is written as `.^`, while a third ancestor is written as `.^^^`
 - a *named member path segment* changes the context to a named member of the current context; for example: `.name` or `."some-member"`
 - an *indexed member path segment* changes the context to an indexed member of the current context, with the index parameters being either (optionally typed) simple values or references; for example: `[1]`; `["ten"]`; `[2,4]`; `[<float>34]`; `[@SOME\_INDEX]`
 - a *collection element path segment* changes the context to a collection element of a specific integer index, provided as a number literal or a binary literal; for example: `[#1]` or `[#0x10]`

There is no way to access a constructor parameter or an index parameter of the current context using path segments. If referencing a parameter is necessary, making it globally identified might be needed.

Also, an indexed member path segment can act as a collection element path segment. It happens when its sole parameter is an implicitly typed number or binary value, and member initialization defines no similarly structured indices. For example, the value of x member will be:

 - "foo", in case of `{ x: $[0] }[ "foo" ]`, `{ x: $[0x0], [0]: "bar" }[ "foo" ]` or `{ x: $[0], [<int>0]: "bar" }[ "foo" ]`
 - "bar", in case of `{ x: $[0], [0]: "bar" }[ "foo" ]` or `{ x: $[<int>0], [<int>0]: "bar" }[ "foo" ]`
 - invalid, in case of `{ x: $[0], [1]: "bar" }[ "foo" ]` or `{ x: $[<int>0], [0]: "bar" }[ "foo" ]`
 
Correct usage of reference addresses, while mostly intuitive, comes with a few caveats. They are yet to be described in more detail in other documents, in particular the formal STON specification.

Example of a valid (if somewhat academic) reference address, obtained by combining together initial context and path segments:

    ^^.items[# 3]["index", @CHILD_OF_INDEX.^]."is_valid?"

Extensions
----------

While STON already has numerous language features, there still might be some potential additions, that would make the documents easier to manage or provide additional data to the applications. For such cases, **extensions** are allowed. Some might be handled at the time of parsing and validating a STON document; these are called *document-side extensions*. Others instead might be processed later by the application, after the document is already built; these are called *application-side extensions*. There are two mechanisms of extending STON documents - *extension types* (as opposed to *regular types*) and *extension members* (as opposed to *regular members*).

An **extension type** is a special kind of named type, indicating that the given entity should be processed differently from the others (for example, it might carry document metadata, or represent some command). All extension types have exclamation mark before their names in STON representation. More specifically, `<!extension>` or `<!"extension">` both represent an extension type, but `<"!extension">` does not.

In general, extension types should be designed in such a way that their function is consistent no matter where they appear in the document, or even what application they are used for.

An **extension member** can be defined in an entity's *member initialization*, just like regular members; however, like extension type, it is handled in a significantly different manner. All extension members have exclamation mark before their names. More specifically, `{ !extension: null }` or `{ !"extension": null }` both represent entities with an extension member, but `{ "!extension": null }` does not. Although an entity cannot define two regular members with the same name, it can define an extension member and regular member sharing a name, and those members are treated as distinct.

Some extension members might be valid only in the context of specific extension types, others might apply to regularly typed entities as well. If an extension member can be applied to regular members, it should be designed in such a way that it acts similarly no matter what type of entity it appears in, regular types and extension types alike.

There might be STON extensions so general and useful that they are worth spreading. In such case, it is strongly recommended that the STON extension is carefully documented and standarized as soon as possible, so that independent extension implementations act consistently. This is particularly important for document-side extensions.

Readability and maintenance
---------------------------

Aside from STON general functionality, there are small but handy syntactic features that make STON documents easier to work with. Some of these were shown earlier, such as string literal chains.

Like JSON, STON allows using so-called *insignificant whitespace* that is omitted when processing the document itself; it can be used to space the document for better readability. The whitespace is not omitted in two cases. First, when it follows a CANUN name and precedes another CANUN name, a number literal or a binary literal; in such case, a whitespace sequence is treated as a single space rather than ignored entirely. Second, when it appears inside a string literal; in such case, it is left untouched.

Unlike JSON, STON allows comments as well. There are two kinds of comments, typical to C-family programming languages. A *single line comment* begins with two slashes and ends at the nearest newline sequence, or the end of file. A *block comment* begins with a slash and asterisk sequence, and ends at the nearest asterisk and slash sequence. Both kinds of comments behave like a whitespace, including the significance rules.
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


Also, STON allows *list-terminating commas*. It means that in any comma-separated list, a single comma might or might not be added right after the last element. It might streamline a workflow a little, whether by removing unnecessary delays caused by minor syntax errors or making the lists easier to manipulate (in particular, when commenting out last elements).

Other resources
===============

There are no other resources available for Specifically Typed Object Notation yet. Formal specification, language reference, tutorials and developers guide are planned for the nearest future (with a stress on the formal specification, in order to clarify the requirements and avoid implementations inconsistency). When it comes to implementations, .NET currently takes the top priority unless otherwise requested.

Credits
=======

The STON language has been designed by Alice Jankowska, also known as Alphish. This language overview has also been written by her.