# Hyperdata

This specification defines Hyperdata, an extension to JSON and IRI that adds classes, IRI identifiers, namespaces, and shared schemas to JSON objects. It is designed to enhance interoperability, data linking, and the self-descriptiveness of JSON data.

## Introduction
Hyperdata is designed to layer on simply to existing JSON data, by adding a `@class`:
```
{
  "@class": "https://example.com/schema#Article",
  "title": "Introduction to Hyperdata",
}
```

Doing this specifies the following values via implicit derivation:
```
object class IRI:             https://example.com/schema#Article       (full-IRI)
namespace IRI:                https://example.com/schema#              (ns-IRI)
object property IRI:          https://example.com/schema#title         (full-IRI)
schema document IRI:          https://example.com/schema               (base-IRI)
short form name-fragments:    Article, title                           (name-fragment)
```
The expectation is that when you dereference the schema document `base-IRI` of `https://example.com/schema` (for example via HTTP GET), a successful response will be a Hyperdata document describing the schema within the established `https://example.com/schema#` namespace.

The dereferenced schema document is expected to specify:
- the `Article` class, described by an object with an `@id` value of `https://example.com/schema#Article`, and
- the named properties of the `Article` class, such as `title`, which would be described by an object with an `@id` value of `https://example.com/schema#title`.

In this manner, the `Article` class and it's `title` property are universally and unambiguously named within the IRI space, allowing shared understanding of readily accessible hyperdata.

By adding an `@id` to the previous example, it can also be universally named within the IRI space, allowing it to be referenced and dereferenced within the web of hyperdata.
```
{
  "@namespace": "https://example.com/schema#",
  "@class": "Article",
  "@id": "https://example.com/hyperdata-introduction#description",
  "title": "Introduction to Hyperdata",
}
```
Again, the expectation is that when you dereference the implicitly derived document `base-IRI` of `https://example.com/hyperdata-introduction`, a successful response will include the above example hyperdata object. This example introduces all three JSON property extensions defined by hyperdata: `@namespace`, `@class`, `@id`, and the short form name-fragment style.

## Basic Concepts

### Hyperdata Document

A Hyperdata Document is a JSON document that can be dereferenced using a unique identifier known as the `base-IRI`.

When accessed over HTTP, the document should be served with the media type `application/json`.
- `base-IRI` - a dereferencable IRI that does not include a `#name-fragment`.
  - `https://example.com/schema`
  
### Hyperdata Namespace

In a Hyperdata Document, the namespace it provides is identified by a `ns-IRI`. This defines a `@namespace` within which Things, Properties, and Classes are named and described.
- `ns-IRI` - a namespace IRI which always ends with a `#`.
  - `https://example.com/schema#`

### Hyperdata Names

In Hyperdata, Things, Properties, or Classes are given canonical `full-IRI` names.
- `full-IRI` - an IRI that ends with a `#name-fragment`.
  - `https://example.com/schema#Person`, `https://example.com/schema#name`

Within a Hyperdata document, a short form `name-fragment` can be used as a more concise way to denote a `full-IRI` when combined with a `@namespace`.
- `name-fragment` - a shorthand syntax used within hyperdata documents to represent a `full-IRI`.
  - `{"@namespace": "https://example.com/schema#", "@class": "Person", "name": "Jon Doe"}`

### Implicit Derivation of Namespaces and Documents

Hyperdata introduces a strict subset of IRI forms, which enable implicit derivation of both the namespace's `ns-IRI`, and the hyperdata document's `base-IRI` from a single `full-IRI`.

Given a `@class` value of `https://example.com/schema#Person`, we can implicitly determine the `@namespace` to be `https://example.com/schema#` and the hyperdata document's dereferenceable `base-IRI` to be `https://example.com/schema`.

Consequently, `{"@namespace": "https://example.com/schema#", "@class": "Person", "name": "Jon Doe"}` can succinctly be represented as `{"@class": "https://example.com/schema#Person", "name": "Jon Doe"}`.

To derive a `ns-IRI` from a `full-IRI`, remove everything after the `#`.
- `https://example.com/schema#Person` to `https://example.com/schema#`

To derive a `base-IRI` from a `full-IRI` or an `ns-IRI`, remove the `#` and any characters following it.
- `https://example.com/schema#Person` OR `https://example.com/schema#` to `https://example.com/schema`

To combine a `full-IRI` from a `ns-IRI` and a `name-fragment`, append the `name-fragment` to the `ns-IRI`.
- `https://example.com/schema#` + `Person` = `https://example.com/schema#Person`

## JSON Property Extensions

### @class
- **Value Space**: `full-IRI`
- **Lexical Space**: `name-fragment` OR `full-IRI`
- **Description**: Specifies the full-IRI of the class of this object.
- **Behavior**: 
  - If the value of `@class` is a `full-IRI`, the value of `@namespace` MUST be set to the computed `ns-IRI`.
    - Example: `"@class": "https://example.com/schema#Person` entails `"@namespace": "https://example.com/schema#"`
  - If the value of `@class` is an `name-fragment`, the value MUST be appened to the in-scope `@namespace` to compute it's `full-IRI` value.
    - Example: `{"@namespace": "https://example.com/schema#", "@class": "Person"}` entails a `full-IRI` of `https://example.com/schema#Person`
 
### @namespace
- **Value Space**: `ns-IRI`
- **Lexical Space**: `#` OR `ns-IRI`
- **Description**: Specifies the namespace of the current object and its children, unless they define a different namespace explicitly via `@namespace` or implicitly via a `full-IRI` `@type`.
- **Behavior**: If not defined on the current in-scope object, it must be inherited from the parent object.

### @id
- **Value Space**: `ns-IRI` OR `base-IRI` OR `full-IRI`
- **Lexical Space**: `ns-IRI` OR `base-IRI` OR `full-IRI` OR `#` OR (`#` `name-fragment`)
- **Description**: Establishes a unique universally quantified `IRI` for the object of which this property belongs, and which the object describes.
- **Behavior**: If not defined on the current in-scope object, the object should be considered as existentially quantified.

## Object Property Name Expansion
- **Value Space**: `full-IRI`
- **Lexical Space**: `name-fragment` OR `full-IRI`
- **Description**: Potentially establishes a unique universally quantified `full-IRI` for the property.
- **Behavior**:
  - If the `full-IRI` cannot be determined to have a hyperdata description after successfully dereferencing the hyperdata document to which it belongs, it should be treated as traditional json name.
  - If the `full-IRI` belongs to a different hyperdata namespace to the current in-scope `@namespace`, it's usage is undefined by this specification.
  - If the `full-IRI` belongs to a different `@class` than the one specified for the in-scope object, it's usage is undefined by this specification.


### Example Hyperdata Document

```
[
  {
   "@namespace": "https://example.com/schema#",
   "@class": "Person",
   "@id": "https://jondoe.example.com/#me",
   "name": "Jon Doe",
   "homepage": {
     "href": "https://jondoe.example.com/",
     "title": "Jon Doe's Homepage"
   },
   "knows":  {
     "@id": "https://janedoe.example.com/#me",
     "name": "Jane Doe"
    }
  },
  ...
]
```

## IRI ABNF Extensions
```
   base-IRI       = scheme ":" ihier-part [ "?" iquery ]
   
   ns-IRI         = base-IRI "#"
   
   full-IRI       = ns-IRI "#" name-fragment
   
   name-fragment  = 1*( ipchar / "/" / "?" )
```

