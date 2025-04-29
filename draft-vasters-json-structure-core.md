---
title: "JSON Structure: Core"
category: info

docname: draft-vasters-json-structure-core-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date: 2025-03-24
consensus: true
v: 3
area: AREA
workgroup: TBD
keyword:
 - JSON
 - schema
venue:
  group: TBD
  type: Working Group
  mail: TBD
  arch: TBD
  github: "json-structure/core"
  latest: "https://json-structure.github.io/core/draft-vasters-json-structure-core.html"

author:
  -
    fullname: Clemens Vasters
    organization: Microsoft Corporation
    email: clemensv@microsoft.com

normative:
  RFC1950:
  RFC1951:
  RFC1952:
  RFC3339:
  RFC3986:
  RFC4122:
  RFC4648:
  RFC6838:
  RFC6901:
  RFC7932:
  RFC8259:
  RFC8949:


informative:
  JSTRUCT-ALTNAMES:
    title: "JSON Structure Alternate Names"
    author:
      - fullname: Clemens Vasters
    target: https://json-structure.github.io/alternate-names
  JSTRUCT-COMPOSITION:
    title: "JSON Structure Conditional Composition"
    author:
      - fullname: Clemens Vasters
    target: https://json-structure.github.io/conditional-composition
  JSTRUCT-IMPORT:
    title: "JSON Structure Import"
    author:
      - fullname: Clemens Vasters
    target: https://json-structure.github.io/import
  JSTRUCT-UNITS:
    title: "JSON Structure Units"
    author:
      - fullname: Clemens Vasters
    target: https://json-structure.github.io/units
  JSTRUCT-VALIDATION:
    title: "JSON Structure Validation"
    author:
      - fullname: Clemens Vasters
    target: https://json-structure.github.io/validation


--- abstract

This document specifies _JSON Structure_, a data structure definition language
that enforces strict typing, modularity, and determinism. _JSON Structure_
describes JSON-encoded data such that mapping to and from programming languages
and databases and other data formats is straightforward.

--- middle

# Introduction {#introduction}

This document specifies _JSON Structure_, a data structure definition language
that enforces strict typing, modularity, and determinism. _JSON Structure_
documents (schemas) describe JSON-encoded data such that mapping JSON encoded
data to and from programming languages and databases and other data formats
becomes straightforward.

_JSON Structure_ is extensible, allowing additional features to be layered on
top. The core language is a data-definition language.

The "Validation" and "Conditional Composition" extension specifications add
rules that allow for complex pattern matching of _JSON Structure_ documents
against JSON data for document validation purposes.

Complementing _JSON Structure_ are a set of extension specifications that extend
the core schema language with additional, OPTIONAL features:

- _JSON Structure: Import_ {{JSTRUCT-IMPORT}}: Defines a mechanism for importing
  external schemas and definitions into a schema document.
- _JSON Structure: Alternate Names and Descriptions_ {{JSTRUCT-ALTNAMES}}:
  Provides a mechanism for declaring multilingual descriptions, and alternate
  names and symbols for types and properties.
- _JSON Structure: Symbols, Scientific Units, and Currencies_ {{JSTRUCT-UNITS}}:
  Defines annotation keywords for specifying symbols, scientific units, and
  currency codes complementing type information.
- _JSON Structure: Validation_ {{JSTRUCT-VALIDATION}}: Specifies extensions to
  the core schema language for declaring validation rules for JSON data that
  have no structural impact on the schema.
- _JSON Structure: Composition_ {{JSTRUCT-COMPOSITION}}: Defines a set of
  conditional composition rules for evaluating schemas.


These extension specifications are enabled by the extensibility
({{extensions-and-add-ins}}) features and can be applied to meta-schemas,
schemas, and JSON document instances.

# Conventions {#conventions}

{::boilerplate bcp14}

# JSON Structure Core Specification {#json-structure-core-specification}

## Schema Elements {#schema-elements}

### Schema {#schema}

A "schema" is a JSON object that describes, constrains, and interprets a JSON
node.

This schema constrains a JSON node to be of type `string`:

~~~ json
{
  "name": "myname",
  "type": "string"
}
~~~

In the case of a schema that references a compound type (`object`, `set`,
`array`, `map`, `tuple`, `choice`), the schema further describes the structure
of the compound type. Schemas can be placed into a namespace ({{namespaces}})
for reuse in other schemas.

~~~ json
{
  "name": "myname",
  "type": "object",
  "properties": {
    "name": { "type": "string" }
  }
}
~~~

All schemas have an associated name that serves as an identifier. In the example
above where the schema is a root object, the name is the value of the `name`
property.

When the schema is placed into a namespace ({{namespaces}}) or embedded into a
properties ({{properties-keyword}}) section of an `object` type, the name is the
key under which the schema is stored.

Further rules for schemas are defined in {{type-system-rules}}.

A "schema document" is a schema that represents the root of a schema hierarchy
and is the container format in which schemas are stored on disk or exchanged. A
schema document MAY contain multiple type declarations and namespaces. The
structure of schema documents is defined in {{document-structure}}.

JSON Structure is extensible. All keywords that are not explicitly defined in
this document MAY be used for custom annotations and extensions. This also
applies to keywords that begin with the `$` character. A complete list of
reserved keywords is provided in {{reserved-keywords}}.

The semantics of keywords defined in this document MAY be expanded by extension
specifications, but the core semantics of the keywords defined in this document
MUST NOT be altered.

Be mindful that the use of custom keywords and annotations might conflict with
future versions of this specification or other extensions and that the authors
of this specification will not go out of their way to avoid such conflicts.

{{extensions-and-add-ins}} details the extensibility features.

Formally, a schema is a constrained non-schema ({{non-schema}}) that requires a
type ({{type-keyword}}) keyword or a `$ref` ({{ref-keyword}}) keyword to be a
schema.

### Non-Schema {#non-schema}

Non-schemas are objects that do not declare or refer to a type. The root of a
schema document ({{document-structure}}) is a non-schema unless it contains a
`type` keyword.

A namespace is a non-schema that contains type declarations and other
namespaces.

### Meta-Schemas {#meta-schemas}

A meta-schema is a schema document that defines the structure and constraints of
another schema document. Meta-schemas are used to validate schema documents and
to ensure that schemas are well-formed and conform to the JSON Structure
specification.

The meta-schemas for JSON Structure and the extension specifications are
enumerated in the Appendix: Metaschemas ({{schema}}).

Meta-schemas can extend existing meta-schemas by adding new keywords or
constraints. The `$schema` keyword is used to reference the meta-schema that a
schema document conforms to, the `$id` keyword is used to define the identifier
of the new meta-schema, and the `$import` keyword defined in the
{{JSTRUCT-IMPORT}} extension specification is used to import all definitions
from the foundational meta-schema.

## Data Types {#data-types}

The data types that can be used with the `type` keyword are categorized into
JSON primitive types, extended types, compound types, and reusable types
{{reusable-types}}.

While JSON Structure builds on the JSON data type model, it introduces a rich
set of types to represent structured data more accurately and to allow more
precise integration with common data types used in programming languages and
data formats. All these extended types have a well-defined representation in
JSON primitive types.

### JSON Primitive Types {#json-primitive-types}

These types map directly to the underlying JSON representation:

#### `string` {#string}

A sequence of Unicode characters enclosed in double quotes.

- Base type: `string` {{Section 7 of RFC8259}}
- Annotations: The `maxLength` keyword can be used on a schema with the `string`
  type to specify the maximum length of the string. By default, the maximum
  length is unlimited. The purpose of the keyword is to inform consumers of the
  maximum space required to store the string.

#### `number` {#number}

A numeric literal without quotes.

- Base type: `number` {{Section 6 of RFC8259}}

Note that the `number` representation in JSON is a textual representation of a
decimal number (base-10) and therefore cannot accurately represent all possible
values of IEE754 floating-point numbers (base-2), in spite of JSON `number`
leaning on the IEEE754 standard as a reference for the value space.

#### `boolean` {#boolean}

A literal `true` or `false` (without quotes).

- Base type: `boolean` {{Section 3 of RFC8259}}

#### `null` {#null}

A literal `null` (without quotes).

- Base type: `null` {{Section 3 of RFC8259}}

### Extended Primitive Types {#extended-primitive-types}

Extended types impose additional semantic constraints on the underlying JSON
types. These types are used to represent binary data, high-precision numeric
values, date and time information, and structured data.

Large integer and decimal types are used to represent high-precision numeric
values that exceed the range of IEEE 754 double-precision format, which is the
foundation for the `number` type in JSON. Per {{Section 6 of RFC8259}},
interoperable JSON numbers have a range of -2⁵³ to 2⁵³–1, which is less than the
range of 64-bit and 128-bit values. Therefore, the `int64`, `uint64`, `int128`,
`uint128`, and `decimal` types are represented as strings to preserve precision.

The syntax for strings representing large integer and decimal types is based on
the {{Section 6 of RFC8259}} syntax for integers and decimals:

- integer = `[minus] int`
- decimal = `[minus] int frac`

#### `binary` {#binary}

A binary value. The default encoding is Base64 {{RFC4648}}. The type annotation
keywords `contentEncoding`, `contentCompression`, and `contentMediaType` can be
used to specify the encoding, compression, and media type of the binary data.

- Base type: `string`
- Constraints:
  - The string value MUST be an encoded binary value, with the encoding
    specified in the `contentEncoding` keyword.

#### `int8` {#int8}

An 8-bit signed integer.

- Base type: `number`
- Constraints:
  - The numeric literal MUST be in the range -2⁷ to 2⁷–1.
  - No decimal points or quotes are allowed.

#### `uint8` {#uint8}

An 8-bit unsigned integer.

- Base type: `number`
- Constraints:
  - The numeric literal MUST be in the range 0 to 2⁸–1.
  - No decimal points or quotes are allowed.

#### `int16` {#int16}

A 16-bit signed integer.

- Base type: `number`
- Constraints:
  - The numeric literal MUST be in the range -2¹⁵ to 2¹⁵–1.
  - No decimal points or quotes are allowed.

#### `uint16` {#uint16}

A 16-bit unsigned integer.

- Base type: `number`
- Constraints:
  - The numeric literal MUST be in the range 0 to 2¹⁶–1.
  - No decimal points or quotes are allowed.

#### `int32` {#int32}

A 32-bit signed integer.

- Base type: `number`
- Constraints:
  - The numeric literal MUST be in the range -2³¹ to 2³¹–1.
  - No decimal points or quotes are allowed.

#### `uint32` {#uint32}

A 32-bit unsigned integer.

- Base type: `number`
- Constraints:
  - The numeric literal MUST be in the range 0 to 2³²–1.
  - No decimal points or quotes are allowed.

#### `int64` {#int64}

A 64-bit signed integer.

- Base type: `string`
- Constraints:
  - The string MUST conform to the {{Section 6 of RFC8259}} definition for the
    `[minus] int` syntax.
  - The string value MUST represent a 64-bit integer in the range -2⁶³ to 2⁶³–1.

#### `uint64` {#uint64}

A 64-bit unsigned integer.

- Base type: `string`
- Constraints:
  - The string MUST conform to the {{Section 6 of RFC8259}} definition for the
    `int` syntax.
  - The string value MUST represent a 64-bit integer in the range 0 to 2⁶⁴–1.

#### `int128` {#int128}

A 128-bit signed integer.

- Base type: `string`
- Constraints:
  - The string MUST conform to the {{Section 6 of RFC8259}} definition for the
    `[minus] int` syntax.
  - The string value MUST represent a 128-bit integer in the range -2¹²⁷ to
    2¹²⁷–1.

#### `uint128` {#uint128}

A 128-bit unsigned integer.

- Base type: `string`
- Constraints:
  - The string MUST conform to the {{Section 6 of RFC8259}} definition for the
    `int` syntax.
  - The string value MUST represent a 128-bit integer in the range 0 to 2¹²⁸–1.

#### `float8` {#float8}

An 8-bit floating-point number.

- Base type: `number`
- Constraints:
  - Conforms to IEEE 754 single-precision value range limits (8 bits), which are
    3 bits of significand and 4 bits of exponent, with a range of approximately
    ±3.4×10³.

#### `float` {#float}

A single-precision floating-point number.

- Base type: `number`
- Constraints:
  - Conforms to IEEE 754 single-precision value range limits (32 bits), which
    are 24 bits of significand and 8 bits of exponent, with a range of
    approximately ±3.4×10³⁸.

IEEE754 binary32 are base-2 encoded and therefore cannot represent all decimal
numbers accurately, and vice versa. In cases where you need to encode IEEE754
values precisely, store the IEE754 binary32 value as an `int32` or `uint32`
number.

#### `double` {#double}

A double-precision floating-point number.

- Base type: `number`
- Constraints:
  - Conforms to IEEE 754 double-precision value range limits (64 bits), which
    are 53 bits of significand and 11 bits of exponent, with a range of
    approximately ±1.7×10³⁰⁸.

IEEE754 binary64 are base-2 encoded and therefore cannot represent all decimal
numbers accurately, and vice versa. In cases where you need to encode IEEE754
values precisely, store the IEE754 binary64 value as an `int64` or `uint64`
number.

#### `decimal` {#decimal}

A decimal number supporting high-precision values.

- Base type: `string`
- Constraints:
  - The string value MUST conform to the {{Section 6 of RFC8259}} definition
    for the `[minus] int frac` syntax.
  - Defaults: 34 significant digits and 7 fractional digits, which is the
    maximum precision supported by the IEEE 754 decimal128 format.
- Annotations:
  - The `precision` keyword MAY be used to specify the total number of
    significant digits.
  - The `scale` keyword MAY be used to specify the number of fractional digits.

#### `date` {#date}

A date in YYYY-MM-DD form.

- Base type: `string`
- Constraints:
  - The string value MUST conform to the {{RFC3339}} `full-date` format.

#### `datetime` {#datetime}

A date and time value with time zone offset.

- Base type: `string`
- Constraints:
  - The string value MUST conform to the {{RFC3339}} `date-time` format.

#### `time` {#time}

A time-of-day value.

- Base type: `string`
- Constraints:
  - The string value MUST conform to the {{RFC3339}} `time` format.

#### `duration` {#duration}

A time duration.

- Base type: `string`
- Constraints:
  - The string value MUST conform to the {{RFC3339}} `duration` format.

#### `uuid` {#uuid}

A universally unique identifier.

- Base type: `string`
- Constraints:
  - The string value MUST conform to the {{RFC4122}} `UUID` format.

#### `uri` {#uri}

A URI reference, relative or absolute.

- Base type: `string`
- Constraints:
  - The string value MUST conform to the {{RFC3986}} `uri-reference` format.

#### `jsonpointer` {#jsonpointer}

A JSON Pointer reference.

- Base type: `string`
- Constraints:
  - The string value MUST conform to the {{RFC6901}} JSON Pointer format.

### Compound Types {#compound-types}

Compound types are used to structure related data elements. JSON Structure
supports the following compound types:

#### `object` {#object}

The `object` type is used to define structured data with named properties. It's
represented as a JSON object, which is an unordered collection of key–value
pairs.

The `object` type MUST include a `name` attribute that defines the name of the
type.

The `object` type MUST include a `properties` attribute that defines the
properties of the object. The `properties` attribute MUST be a JSON object where
each key is a property name and each value is a schema definition for the
property. The object MUST contain at least one property definition.

The `object` type MAY include a `required` attribute that defines the required
properties of the object.

The `object` type MAY include an `additionalProperties` attribute that defines
whether additional properties are allowed and/or what their schema is.

Example:

~~~ json
{
  "name": "Person",
  "type": "object",
  "properties": {
    "name": { "type": "string" },
    "age": { "type": "int32" }
  },
  "required": ["name"],
  "additionalProperties": false
}
~~~

#### `array` {#array}

An `array` type is used to define an ordered collection of elements. It's
represented as a JSON array, which is an ordered list of values.

The `items` attribute of an array MUST reference a reusable type or a primitive
type or a locally declared compound type.

**Examples:**

~~~ json
{
  "type": "array",
  "items": { "type": { "$ref": "#/Namespace/TypeName" } }
}
~~~

~~~ json
{
  "type": "array",
  "items": { "type": "string" }
}
~~~

#### `set` {#set}

The `set` type is used to define an unordered collection of unique elements.
It's represented as a JSON array where all elements are unique.

The `items` attribute of a `set` MUST reference a reusable type or a primitive
type or a locally declared compound type.

Example:

~~~ json
{
  "type": "set",
  "items": { "$ref": "#/Namespace/TypeName" }
}
~~~

~~~ json
{
  "type": "set",
  "items": { "type": "string" }
}
~~~

#### `map` {#map}

The `map` type is used to define dynamic key–value pairs. It's represented as a
JSON object where the keys are strings and the values are of a specific type.

All keys in a `map` MUST conform to the identifier rules ({{identifier-rules}}).

The `values` attribute of a `map` MUST reference a reusable type or a primitive
type or a locally declared compound type.

Example:

~~~ json
{
  "type": "map",
  "values": { "$ref": "#/StringType" }
}
~~~

#### `tuple` {#tuple}

The `tuple` type is used to define an ordered collection of elements with a
specific length. It's represented as a JSON array where each element is of a
specific type.

The elements are defined using a `properties` map as with the `object`
({{object}}) type and each element is named. This permits straightforward
mapping into application constructs. All declared properties of a `tuple` are
implicitly REQUIRED.

The order of the elements in a tuple is declared using the `tuple` keyword
{{tuple-keyword}}, which is REQUIRED. The `tuple` keyword MUST be a JSON array
of strings, where each declared property name MUST be an element of the array.
The order of the elements in the array defines the order of the properties in
the tuple.

A `tuple` type MUST include a `name` attribute that defines the name of the
type.

Example:

~~~ json
{
  "type": "tuple",
  "name": "Person",
  "properties": {
    "name": { "type": "string" },
    "age": { "type": "int32" }
  },
  "tuple": ["name", "age"]
}
~~~

The following JSON node is an valid instance of the `tuple` type defined above:

~~~ json
["Alice", 42]
~~~

#### `any` {#any}

The `any` type is used to define a type that can be any JSON value, including
primitive types, compound types, and extended types.

Example:

~~~ json
{
  "type": "any"
}
~~~

#### `choice` {#choice}

The `choice` type is used to define a "discriminated union" of types. A
choice is a set of types where only one type can be selected at a time and
where the selected type is determined by the value of a selector.

The `choice` type can declare two variants of discriminated unions that
are represented differently in JSON:

- _Tagged unions_: The `choice` type is represented as a JSON object with a
  single property whose name is the selector of the type as declared in the
  `choices` ({{choices-keyword}}) map and whose value is of the selected type.
- _Inline unions_: The `choice` type is represented as a JSON object of the
  selected type with the selector as a property of the object.

##### Tagged Unions {#tagged-unions}

A tagged union is declared as follows:

~~~ json
{
  "type": "choice",
  "name": "MyChoice",
  "choices": {
    "string": { "type": "string" },
    "int32": { "type": "int32" }
  }
}
~~~

The JSON node described by the schema above is a tagged union. For the
example, the following JSON node is a valid instance of the `MyChoice` type:

~~~ json
{
  "string": "Hello, world!"
}
~~~

or:

~~~ json
{
  "int32": 42
}
~~~

##### Inline Unions {#inline-unions}

Inline unions require for all type choices to extend a common base type.

This is expressed by using the `$extends` ({{extends-keyword}}) keyword in the
`choice` declaration. The `$extends` keyword MUST refer to a schema that defines
the base type and the base type MUST be abstract.

If `$extends` is present, the `selector` property declares the name of the
injected property that acts as the selector for the inline union. The
type of the selector property is `string`. The selector property MAY
shadow a property of the base type; in this case, the base type property
MUST be of type `string`.

The selector is defined as a property of the base type and the value of the
selector property MUST be a string that matches the name of one of the
options in the `choices` map.

Example:

~~~ json
{
  "$schema": "https://json-structure.org/meta/core/v0/#",
  "$id": "https://schemas.vasters.com/TypeName",
  "type": "choice",
  "$extends": "#/definitions/Address",
  "selector": "addressType",
  "choices": {
    "StreetAddress": { "$ref": "#/definitions/StreetAddress" },
    "PostOfficeBoxAddress": { "$ref": "#/definitions/PostOfficeBoxAddress" }
  },
  "definitions" : {
    "Address": {
      "abstract": true,
      "type": "object",
      "properties": {
          "city": { "type": "string" },
          "state": { "type": "string" },
          "zip": { "type": "string" }
      }
    },
    "StreetAddress": {
      "type": "object",
      "$extends": "#/definitions/Address",
      "properties": {
          "street": { "type": "string" }
      }
    },
    "PostOfficeBoxAddress": {
      "type": "object",
      "$extends": "#/definitions/Address",
      "properties": {
          "poBox": { "type": "string" }
      }
    }
  }
}
~~~

The JSON node described by the schema above is an inline union. This
example shows a JSON node that is a street address:

~~~ json
{
  "addressType": "StreetAddress",
  "street": "123 Main St",
  "city": "Seattle",
  "state": "WA",
  "zip": "98101"
}
~~~

This example shows a JSON node that is a post office box address:

~~~ json
{
  "addressType": "PostOfficeBoxAddress",
  "poBox": "1234",
  "city": "Seattle",
  "state": "WA",
  "zip": "98101"
}
~~~


## Document Structure {#document-structure}

A JSON Structure document is a JSON object that contains schemas ({{schema}})

The root of a JSON Structure document MUST be a JSON object.

The root object MUST contain the following REQUIRED keywords:

- `$id`: A URI that is the unique identifier for this schema document.
- `$schema`: A URI that identifies the version and meta-schema of the JSON
  Structure specification used.
- `name`: A string that provides a name for the document. If the root object
   defines a type, the `name` attribute is also the name of the type.

The presence of both keywords identifies the document as a JSON Structure
document.

The root object MAY contain the following OPTIONAL keywords:

- `$root`: A JSON Pointer that designates a reusable type as the root type for
  instances.
- `definitions`: The root of the type declaration namespace hierarchy.
- `type`: A type declaration for the root type of the document. Mutually
  exclusive with `$root`.
- if `type` is present, all annotations and constraints applicable to this
  declared root type are also permitted at the root level.

### Namespaces {#namespaces}

A namespace is a JSON object that provides a scope for type declarations or
other namespaces. Namespaces MAY be nested within other namespaces.

The `definitions` keyword forms the root of the namespace hierarchy for reusable
type definitions. All type declarations immediately under the `definitions`
keyword are in the root namespace.

A `type` definition at the root is placed into the root namespace as if it were
a type declaration under `definitions`.

Any object in the `definitions` map that is not a type declaration is a
namespace.

Example with inline `type`:

~~~ json
{
    "$schema": "https://json-structure.org/meta/core/v0/#",
    "$id": "https://schemas.vasters.com/TypeName",
    "name": "TypeName",
    "type": "object",
    "properties": {
        "name": { "type": "string" }
    }
}
~~~

Example with `$root`:

~~~ json
{
    "$schema": "https://json-structure.org/meta/core/v0/#",
    "$id": "https://schemas.vasters.com/TypeName",
    "$root": "#/definitions/TypeName",
    "definitions": {
        "TypeName": {
            "type": "object",
            "properties": {
                "name": { "type": "string" }
            }
        }
    }
}
~~~

Example with the root type in a namespace:

~~~ json
{
    "$schema": "https://json-structure.org/meta/core/v0/#",
    "$id": "https://schemas.vasters.com/TypeName",
    "$root": "#/definitions/Namespace/TypeName",
    "definitions": {
        "Namespace": {
            "TypeName": {
                "name": "TypeName",
                "type": "object",
                "properties": {
                    "name": { "type": "string" }
                }
            }
        }
    }
}
~~~

### `$schema` Keyword {#schema-keyword}

The value of the REQUIRED `$schema` keyword MUST be an absolute URI. The keyword
has different functions in JSON Structure documents and JSON documents.

- In JSON Structure schema documents, the `$schema` keyword references a
  meta-schema that this document conforms to.
- In JSON documents, the `$schema` keyword references a JSON Structure schema
  document that defines the structure of the JSON document.

The value of `$schema` MUST correspond to the `$id` of the referenced
meta-schema or schema document.

The `$schema` keyword MUST be used at the root level of the document.

Example:

~~~ json
{
    "$schema": "https://json-structure.org/meta/core/v0/#",
    "name": "TypeName",
    "type": "object",
    "properties": {
        "name": { "type": "string" }
    }
}
~~~

Use of the keyword `$schema` does NOT import the referenced schema document such
that its types become available for use in the current document.

### `$id` Keyword {#id-keyword}

The REQUIRED `$id` keyword is used to assign a unique identifier to a JSON
Structure schema document. The value of `$id` MUST be an absolute URI. It SHOULD
be a resolvable URI (a URL).

The `$id` keyword is used to identify a schema document in references like
`$schema`.

The `$id` keyword MUST only be used once in a document, at the root level.

Example:

~~~ json
{
    "$schema": "https://json-structure.org/meta/core/v0/#",
    "$id": "https://schemas.vasters.com/TypeName",
    "name": "TypeName",
    "type": "object",
    "properties": {
        "name": { "type": "string" }
    }
}
~~~

### `$root` Keyword {#root-keyword}

The OPTIONAL `$root` keyword is used to designate any reusable type defined in
the document as the root type of this schema document. The value of `$root` MUST
be a valid JSON Pointer that resolves to an existing type definition inside the
`definitions` object.

The `$root` keyword MUST only be used once in a document, at the root level. Its
use is mutually exclusive with the `type` keyword.

Example:

~~~ json
{
  "$schema": "https://json-structure.org/meta/core/v0/#",
  "$id": "https://schemas.vasters.com/TypeName",
  "$root": "#/definitions/Namespace/TypeName",
  "definitions": {
      "Namespace": {
          "TypeName": {
            "name": "TypeName",
            "type": "object",
            "properties": {
                "name": { "type": "string" }
            }
          }
      }
  }
}
~~~

### `definitions` Keyword {#definitions-keyword}

The `definitions` keyword defines a namespace hierarchy for reusable type
declarations. The keyword MUST be used at the root level of the document.

The value of the `definitions` keyword MUST be a map of types and namespaces.
The namespace at the root level of the `definitions` keyword is the root
namespace.

A namespace is a JSON object that provides a scope for type declarations or
other namespaces. Any JSON object under the `definitions` keyword that is not a
type definition (containing the `type` attribute) is considered a namespace.

Example:

~~~ json
{
    "$schema": "https://json-structure.org/meta/core/v0/#",
    "$id": "https://schemas.vasters.com/TypeName",
    "definitions": {
        "Namespace": {
            "TypeName": {
                "name": "TypeName",
                "type": "object",
                "properties": {
                    "name": { "type": "string" }
                }
            }
        }
    }
}
~~~

### `$ref` Keyword {#ref-keyword}

References to type declarations within the same document MUST use a schema
containing a single property with the name `$ref` as the value of `type`. The
value of `$ref` MUST be a valid JSON Pointer that resolves to an existing type
definition.

Example:

~~~ json
{
  "$schema": "https://json-structure.org/meta/core/v0/#",
  "$id": "https://schemas.vasters.com/TypeName",
  "properties": {
      "name1": { "type": { "$ref": "#/definitions/Namespace/TypeName" }},
      "name2": { "type": { "$ref": "#/definitions/Namespace2/TypeName2" }}
  },
  "definitions": {
      "Namespace": {
          "TypeName": {
              "name": "TypeName",
              "type": "object",
              "properties": {
                  "name": { "type": "string" }
              }
          }
      },
      "Namespace2": {
          "TypeName2": {
              "name": "TypeName2",
              "type": "object",
              "properties": {
                  "name": { "type": { "$ref": "#/definitions/Namespace/TypeName" }}
              }
          }
      }
  }
}
~~~

The `$ref` keyword is only permitted inside the `type` attribute value of a
schema definition, including in type unions.

`$ref` is NOT permitted in other attributes and MUST NOT be used inside the
`type` of the root object.

### `id` Keyword {#object-id-keyword}

The `id` keyword is only applicable on objects and states the properties that
make up the primary ID(s) of the object. Providing more than one ID indicates
a composite primary ID, not a list of alternative IDs.

Example:

~~~ json
{
  "$schema": "https://json-structure.org/meta/core/v0/#",
  "$id": "https://schemas.vasters.com/TypeName",
  "definitions": {
      "Namespace": {
          "TypeName": {
              "name": "TypeName",
              "type": "object",
              "id": ["ID"],
              "properties": {
                "ID": { "type": "string" }
              }
          }
      }
  }
}
~~~

The `id` MUST only be used on objects.

### Cross-references {#cross-references}

In JSON Structure documents, the `$schema` keyword references the meta-schema of
this specification. In JSON documents, the `$schema` keyword references the
schema document that defines the structure of the JSON document. The value of
`$schema` is a URI. Ideally, the URI SHOULD be a resolvable URL to a schema
document, but it's primarily an identifier. As an identifier, it can be used as
a lookup key in a cache or schema-registry.

The OPTIONAL {{JSTRUCT-IMPORT}} extension specification is the exception and
provides a mechanism for importing definitions from external schemas.

## Type System Rules {#type-system-rules}

### Schema Declarations {#schema-declarations}

- Every schema element MUST declare a `type` referring to a primitive, compound,
  or reusable type.
- To reference a reusable type, the `type` attribute MUST be a schema with a
  single `$ref` property resolving to an existing type declaration.
- Compound types SHOULD be declared in the `definitions` section as reusable
  types. Inline compound types in arrays, maps, unions, or property definitions
  MUST NOT be referenced externally.
- Primitive and compound type declarations are confined to this specification.
- Defined types:
  - **JSON Primitives:** `string`, `number`, `boolean`, `null`.
  - **Extended Primitives:** `int32`, `uint32`, `int64`, `uint64`, `int128`,
    `uint128`, `float`, `double`, `decimal`, `date`, `datetime`, `time`,
    `duration`, `uuid`, `uri`, `binary`, `jsonpointer`.
  - **JSON Compounds:** `object`, `array`.
  - **Extended Compounds:** `map`, `set`, `tuple`, `any`, `choice`.

### Reusable Types {#reusable-types}

- Reusable types MUST be defined in the `definitions` section.
- Each declaration in `definitions` MUST have a unique, case-sensitive name
  within its namespace. The same name MAY appear in different namespaces.

### Type References {#type-references}

- Use `$ref` to reference types declared in the same document.
- `$ref` MUST be a valid JSON Pointer to an existing type declaration.
- `$ref` MAY include a `description` attribute for additional context.

### Dynamic Structures {#dynamic-structures}

- Use the `map` type for dynamic key–value pairs. The `object` type requires at
  least one property and cannot model fully dynamic properties with
  `additionalProperties`.
- The `values` attribute of a `map` and the `items` attribute of an `array` or
  `set` MUST reference a reusable type, a primitive type, or a locally declared
  compound type.

## Composition Rules {#composition-rules}

This section defines the rules for composing schemas. Further, OPTIONAL
composition rules are defined in the {{JSTRUCT-COMPOSITION}} extension
specification.

### Unions {#unions}

- Non-discriminated type unions are formed as sets of primitive types and type
  references. It is NOT permitted to define a compound type inline inside a
  non-discriminated type union. Discriminated unions are formed as a
  `choice` ({{choice}}) type to which the rules of this section do not apply.
- A type union is a composite type reference and not a standalone compound type
  and is therefore not named.
- The JSON node described by a schema with a type union MUST conform to at least
  one of the types in the union.
- If the JSON node described by a schema with a type union conforms to more than
  one type in the union, the JSON node MUST be considered to be of the first
  matching type in the union.

**Examples:**

Union of a string and a compound type:

~~~ json
{
  "type": ["string", { "$ref": "#/Namespace/TypeName" } ]
}
~~~

Union of a string and an `int32`:

~~~ json
{
  "type": ["string", "int32"]
}
~~~

A valid union of a string and a `map` of strings:

~~~ json
{
  "type": ["string", { "type": "map", "values": { "type": "string" } } ]
}
~~~

An inline definition of a compound type in a union is NOT permitted:

~~~ json
{
  "type": ["string", { "type": "object", "properties": { "name": { "type": "string" } } } ]
}
~~~

### Prohibition of Top-Level Unions {#prohibition-of-top-level-unions}

- The root of a JSON Structure document MUST NOT be an array.
- If a type union is desired as the type of the root of a document instance, the
  `$root` keyword MUST be used to designate a type union as the root type.

## Identifier Rules {#identifier-rules}

All property names and type names MUST conform to the regular expression
`[A-Za-z_][A-Za-z0-9_]*`. They MUST begin with a letter or underscore and MAY
contain letters, digits, and underscores. Keys and type names are
case-sensitive.

`map` keys MAY additionally contain the characters `.` and `-` and MAY begin
with a digit.

If names need to contain characters outside of this range, consider using the
{{JSTRUCT-ALTNAMES}} extension specification to define those.

## Structural Keywords {#structural-keywords}

### The `type` Keyword {#type-keyword}

Declares the type of a schema element as a primitive or compound type. The
`type` keyword MUST be present in every schema element. For unions, the value of
`type` MUST be an array of type references or primitive type names.

**Example**:

~~~ json
{
  "type": "string"
}
~~~

### The `properties` Keyword {#properties-keyword}

`properties` defines the properties of an `object` type.

The `properties` keyword MUST contain a map of property names mapped to schema
definitions.

**Example**:

~~~ json
{
  "type": "object",
  "properties": {
    "name": { "type": "string" },
    "age": { "type": "int32" }
  }
}
~~~

### The `required` Keyword {#required-keyword}

`required` defines the required properties of an `object` type. The `required`
keyword MUST only be used in schemas of type `object`.

The value of the `required` keyword is a simple array of property names or an
array of arrays of property names.

An array of arrays is used to define alternative sets of required properties.
When alternative sets are used, exactly one of the sets MUST match the
properties of the object, meaning they are mutually exclusive.

Property names in the `required` array MUST be present in `properties`.

Example:

~~~ json
{
  "type": "object",
  "properties": {
    "name": { "type": "string" },
    "age": { "type": "int32" }
  },
  "required": ["name"]
}
~~~

Example with alternative sets:

Because the `name` property is required in both sets, the `name` property is
required in all objects. The `fins` property is required in the first set, and
the `legs` property is required in the second set. That means that an object
MUST have either `fins` or `legs` but not both.

~~~ json
{
  "type": "object",
  "properties": {
    "name": { "type": "string" },
    "fins": { "type": "int32" },
    "legs": { "type": "int32" },
    "wings": { "type": "int32" }
  },
  "required": [["name", "fins"], ["name", "legs"]]
}
~~~

### The `items` Keyword {#items-keyword}

Defines the schema for elements in an `array` or `set` type. The value is a type
reference or a primitive type name or a locally declared compound type.

Examples:

~~~ json
{
  "type": "array",
  "items": { "type": { "$ref": "#/Namespace/TypeName" }}
}
~~~

~~~ json
{
  "type": "array",
  "items": { "type": "string" }
}
~~~

### The `values` Keyword {#values-keyword}

Defines the schema for values in a `map` type.

The `values` keyword MUST reference a reusable type or a primitive type or a
locally declared compound type.

Example:

~~~ json
{
  "type": "map",
  "values": { "type": "string" }
}
~~~

### The `const` Keyword {#const-keyword}

Constrains the values of the JSON node described by the schema to a single,
specific value. The `const` keyword MUST appear only in schemas with a primitive
`type`, and the instance value MUST match the provided constant exactly.

**Example**:

~~~ json
{
  "type": "string",
  "const": "example"
}
~~~

### The `enum` Keyword {#enum-keyword}

Constrains a schema to match one of a specific set of values. The `enum` keyword
MUST appear only in schemas with a primitive `type`, and all values in the enum
array MUST match that type. Values MUST be unique.

**Example**:

~~~ json
{
  "type": "string",
  "enum": ["value1", "value2", "value3"]
}
~~~

It is NOT permitted to use `enum` in conjunction with a type union in `type`.

### The `additionalProperties` Keyword {#additionalproperties-keyword}

`additionalProperties` defines whether additional properties are allowed in an
`object` type and, optionally, what their schema is. The value MUST be a boolean
or a schema. If set to `false`, no additional properties are allowed. If
provided with a schema, each additional property MUST conform to it.

**Example**:

~~~ json
{
  "type": "object",
  "properties": {
    "name": { "type": "string" }
  },
  "additionalProperties": false
}
~~~

### The `choices` Keyword {#choices-keyword}

`choices` defines the choices of a `choice` type. The value MUST be a map of
type names to schemas. Each type name MUST be unique within the `choices` map.

The value of each type name MUST be a schema. Inline compound types are permitted.

The `choices` keyword MUST only be used in schemas of type `choice` ({{choice}}).

***Example**:

~~~ json
{
  "type": "choice",
  "name": "MyChoice",
  "choices": {
    "string": { "type": "string" },
    "int32": { "type": "int32" }
  }
}
~~~


### The `selector` Keyword {#selector-keyword}

The `selector` keyword defines the name of the property that acts as the selector
for the type in a `choice` type. The value of `selector` MUST be a string.

The `selector` keyword MUST only be used in schemas of type `choice` ({{choice}}).

See `choice` ({{choice}}) for an example.

### The `tuple` Keyword {#tuple-keyword}

The `tuple` keyword defines the order of properties in a `tuple` type. The
value of `tuple` MUST be an array of strings, where each string is the name of a
property defined in the `properties` map. The order of the strings in the array
defines the order of the properties in the tuple.

The `tuple` keyword MUST only be used in schemas of type `tuple` ({{tuple}}).

See `tuple` ({{tuple}}) for an example.

## Type Annotation Keywords {#type-annotation-keywords}

Type annotation keywords provide additional metadata about the underlying type.
These keywords are used for documentation and validation of additional
constraints on types.

### The `maxLength` Keyword {#maxlength-keyword}

Specifies the maximum allowed length for a string. The `maxLength` keyword MUST
be used only with `string` types, and the string’s length MUST not exceed this
value.

The purpose of `maxLength` is to provide a known storage constraint on the
maximum length of a string. The value MAY be used for validation.

**Example**:

~~~ json
{
  "type": "string",
  "maxLength": 255
}
~~~

### The `precision` Keyword {#precision-keyword}

Specifies the total number of significant digits for numeric values. The
`precision` keyword is used as an annotation for `number` or `decimal` types.

**Example**:

~~~ json
{
  "type": "decimal",
  "precision": 10
}
~~~

### The `scale` Keyword {#scale-keyword}

Specifies the number of digits to the right of the decimal point for numeric
values. The `scale` keyword is used as an annotation for `number` or `decimal`
types to constrain the fractional part.

**Example**:

~~~ json
{
  "type": "decimal",
  "scale": 2
}
~~~

### The `contentEncoding` Keyword {#contentencoding-keyword}

Specifies the encoding of a binary value. The `contentEncoding` keyword is used
as an annotation for `binary` types.

The permitted values for `contentEncoding` are defined in {{RFC4648}}:

- `base64`: The binary value is encoded as a base64 string.
- `base64url`: The binary value is encoded as a base64url string.
- `base16`: The binary value is encoded as a base16 string.
- `base32`: The binary value is encoded as a base32 string.
- `base32hex`: The binary value is encoded as a base32hex string.

**Example**:

~~~ json
{
  "type": "binary",
  "encoding": "base64"
}
~~~

### The `contentCompression` Keyword {#contentcompression-keyword}

Specifies the compression algorithm used for a binary value before encoding. The
`contentCompression` keyword is used as an annotation for `binary` types.

The permitted values for `contentCompression` are:

- `gzip`: The binary value is compressed using the gzip algorithm. See
  {{RFC1952}}.
- `deflate`: The binary value is compressed using the deflate algorithm. See
  {{RFC1951}}.
- `zlib`: The binary value is compressed using the zlib algorithm. See
  {{RFC1950}}.
- `brotli`: The binary value is compressed using the brotli algorithm. See
  {{RFC7932}}.

**Example**:

~~~ json
{
  "type": "binary",
  "encoding": "base64",
  "compression": "gzip"
}
~~~

### The `contentMediaType` Keyword {#contentmediatype-keyword}

Specifies the media type of a binary value. The `contentMediaType` keyword is
used as an annotation for `binary` types.

The value of `contentMediaType` MUST be a valid media type as defined in
{{RFC6838}}.

**Example**:

~~~ json
{
  "type": "binary",
  "encoding": "base64",
  "mediaType": "image/png"
}
~~~

## Documentation Keywords {#documentation-keywords}

Documentation keywords provide descriptive information for schema elements. They
are OPTIONAL but RECOMMENDED for clarity.

### The `description` Keyword {#description-keyword}

Provides a human-readable description of a schema element. The `description`
keyword SHOULD be used to document any schema element.

**Example**:

~~~ json
{
  "type": "string",
  "description": "A person's name"
}
~~~

For multi-lingual descriptions, the {{JSTRUCT-ALTNAMES}} companion provides an
extension to define several concurrent descriptions in multiple languages.

### The `examples` Keyword {#examples-keyword}

Provides example instance values that conform to the schema. The `examples`
keyword SHOULD be used to document potential instance values.

**Example**:

~~~ json
{
  "type": "string",
  "examples": ["example1", "example2"]
}
~~~

## Extensions and Add-Ins {#extensions-and-add-ins}

The `abstract` and `$extends` keywords enable controlled type extension,
supporting basic object-oriented-programming-style inheritance while not
permitting subtype polymorphism where a sub-type value can be assigned a
base-typed property. This approach avoids validation complexities and mapping
issues between JSON schemas, programming types, and databases.

An _extensible type_ is declared as `abstract` and serves as a base for
extensions. For example, a base type _Address_ MAY be extended by
_StreetAddress_ and _PostOfficeBoxAddress_ via `$extends`, but _Address_ cannot
be used directly.

Example:

~~~ json
{
  "$schema": "https://json-structure.org/meta/core/v0/#",
  "definitions" : {
    "Address": {
      "abstract": true,
      "type": "object",
      "properties": {
          "city": { "type": "string" },
          "state": { "type": "string" },
          "zip": { "type": "string" }
      }
    },
    "StreetAddress": {
      "type": "object",
      "$extends": "#/definitions/Address",
      "properties": {
          "street": { "type": "string" }
      }
    },
    "PostOfficeBoxAddress": {
      "type": "object",
      "$extends": "#/definitions/Address",
      "properties": {
          "poBox": { "type": "string" }
      }
    }
  }
}
~~~

A _add-in type_ is declared as `abstract` and `$extends` a specific type that
does not need to be abstract. For example, an add-in type _DeliveryInstructions_
might be applied to any _StreetAddress_ types in a document:

~~~ json
{
    "$schema": "https://json-structure.org/meta/core/v0/#",
    "$id": "https://schemas.vasters.com/Addresses",
    "$root": "#/definitions/StreetAddress",
    "$offers": {
        "DeliveryInstructions": "#/definitions/DeliveryInstructions"
    },
    "definitions" : {
      "StreetAddress": {
        "type": "object",
        "properties": {
            "street": { "type": "string" },
            "city": { "type": "string" },
            "state": { "type": "string" },
            "zip": { "type": "string" }
        }
      },
      "DeliveryInstructions": {
        "abstract": true,
        "type": "object",
        "$extends": "#/definitions/StreetAddress",
        "properties": {
            "instructions": { "type": "string" }
        }
      }
    }
}
~~~

Add-in types are options that a document author can enable for a schema. The
definitions of add-in types are not part of the main schema by default, but are
injected into the designated schema type when the document author chooses to use
them.

Add-in types are advertised in the schema document through the `$offers`
keyword, which is a map that defines add-in names for add-in schema definitions
that exist in the document.

Add-ins are applied to a schema by referencing the add-in name in the `$uses`
keyword that is available only in instance documents. The `$uses` keyword is a
set of add-in names that are applied to the schema for the document.

~~~ json
{
  "$schema": "https://schemas.vasters.com/Addresses",
  "$uses": ["DeliveryInstructions"],
  "street": "123 Main St",
  "city": "Anytown",
  "state": "QA",
  "zip": "00001",
  "instructions": "Leave at the back door"
}
~~~

### The `abstract` Keyword {#abstract-keyword}

The `abstract` keyword declares a type as abstract. This prohibits its direct
use in any type declaration or as the type of a schema element. Abstract types
are used as base types for extension via `$extends` or as add-in types via
`$addins`.

Abstract types implicitly permit additional properties (`additionalProperties`
is always `true`).

- **Value**: A boolean (`true` or `false`).
- **Rules**:
  - The `abstract` keyword MUST only be used in schemas of type `object` and
    `tuple`.
  - Abstract types MUST NOT be used as the type of a schema element or
    referenced via `$ref`.
  - The `additionalProperties` keyword MUST NOT be used on abstract types (its
    value is implicitly `true`).
  - Abstract types MAY extend other abstract types via `$extends`.

### The `$extends` Keyword {#extends-keyword}

The `$extends` keyword merges all properties from an abstract base type into the
extending type.

If the type using `$extends` is marked as `abstract` and referenced via
`$addins`, the composite type _replaces_ the base type in the type model of the
document.

- **Value**: A JSON Pointer to an abstract type.
- **Rules**:
  - The `$extends` keyword MUST only be used in schemas of type `object` and
    `tuple`.
  - The value of `$extends` MUST be a valid JSON Pointer that points to an
    abstract type within the same document.
  - The extending type MUST merge the abstract type’s properties and constraints
    and MUST NOT redefine any inherited property.

### The `$offers` Keyword {#offers-keyword}

The `$offers` keyword is used to advertise add-in types that are available for
use in a schema document. The `$offers` keyword is a map of add-in names to
add-in schema definitions.

- **Value**: A map of add-in names to add-in schema definitions.
- **Rules**:
  - The `$offers` keyword MUST only be used in the root object of a schema
    document.
  - The value of `$offers` MUST be a map where each key is a string and each
    value is a JSON Pointer to an add-in schema definition in the same document
    or a set of JSON Pointers to add-in schema definitions in the same document.
    If the value is a set, the add-in name selects all add-in schema definitions
    at the same time.
  - The keys in the `$offers` map MUST be unique.

### The `$uses` Keyword {#uses-keyword}

The `$uses` keyword is used to apply add-in types to a schema _in an instance
document_ that references the schema. The keyword MAY be used in a meta-schema
that references a parent schema.

- **Value**: A set of add-in names or JSON Pointers to add-in schema definitions
  in the same meta-schema document.
- **Rules**:
  - The `$uses` keyword MUST only be used in instance documents.
  - The value of `$uses` MUST be a set of strings that are either:
     - add-in names advertised in the `$offers` keyword of the schema document
       referenced by the `$schema` keyword of the instance document or
     - JSON Pointers to add-in schema definitions in the same meta-schema
       document.

# Reserved Keywords {#reserved-keywords}

The following keywords are reserved in JSON Structure and MUST NOT be used as
custom annotations or extension keywords:

- `definitions`
- `$extends`
- `$id`
- `$ref`
- `$root`
- `$schema`
- `$uses`
- `$offers`
- `abstract`
- `additionalProperties`
- `choices`
- `const`
- `default`
- `description`
- `enum`
- `examples`
- `format`
- `items`
- `maxLength`
- `name`
- `precision`
- `properties`
- `required`
- `scale`
- `selector`
- `type`
- `values`

# CBOR Type System Mapping

CBOR {{RFC8949}} is a binary encoding of JSON-like data structures. The CBOR
type system is a superset of the JSON type system and adds "binary strings" as
its most substantial type system extension. Otherwise, CBOR is structurally
compatible with JSON.

JSON Structure MAY be used to describe CBOR-encoded data structures. For
encoding CBOR data structures, the data structure is first mapped to a JSON type
model as described in this specification, with the exception that the {{binary}}
primitive type is preserved as a byte array. The resulting mapping is converted
into CBOR per the rules spelled out in {{Section 6.2 of RFC8949}}.

The decoding process is the reverse of the encoding process. The CBOR-encoded
data structure is first decoded into a JSON type model, and then the JSON type
model is validated against the JSON Structure schema, with `binary` types
validated as byte arrays.

# Media Type {#media-type}

The media type for JSON Structure documents is `application/json-structure`.

It is RECOMMENDED to append the structured syntax suffix `+json` to indicate
unambiguously that the content is a JSON document, if the document is a JSON
document. In spite of this specification being focused on JSON, the JSON
Structure documents MAY be encoded using other serialization formats that can
represent the same data structure, such as CBOR {{RFC8949}}.

 - Type name: application
 - Subtype name: json-structure
 - Required parameters: none
 - Optional parameters: none
 - Encoding considerations: binary
 - Security considerations: see {{security-considerations}}
 - Interoperability considerations: none
 - Published specification: this document
 - Applications that use this media type: none
 - Fragment identifier considerations: none
 - Additional information: none


# Media Type Parameters {#media-type-parameters}

While the media type `application/json-structure` does not have any parameters,
this specification defines a parameter applicable to all JSON documents.

## `schema` Parameter {#schema-parameter}

The `schema` parameter is used to reference a JSON Structure document that
defines the structure of the JSON document. The value of the `schema` parameter
MUST be a URI that references and ideally resolves to a JSON Structure document.

The `schema` parameter MAY be used in conjunction with the `application/json`
media type or the `+json` structured syntax suffix or any other media type that
is known to be encoded as JSON.

Example using the HTTP `Content-Type` header:

~~~ http
Content-Type: application/json; schema="https://schemas.vasters.com/TypeName"
~~~

# Security Considerations {#security-considerations}

JSON Structure documents are self-contained and MUST NOT allow external
references except for the `$schema` and `$addins` keywords. Implementations MUST
ensure that all `$ref` pointers resolve within the same document to eliminate
security vulnerabilities related to external schema inclusion.

# IANA Considerations {#iana-considerations}

IANA shall be requested to register the media type `application/json-structure`
as defined in this specification in the "Media Types" registry.

IANA shall be requested to register the parameter `schema` for the
`application/json` media type in the "Media Type Structured Syntax Suffixes"
registry.

--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.

