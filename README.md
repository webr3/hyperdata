# Hyperdata

This specification defines Hyperdata, an extension to JSON and IRI that adds classes, IRI identifiers, namespaces, and shared schemas to JSON objects. It is designed to enhance interoperability, data linking, and the self-descriptiveness of JSON data.

## Introduction
Hyperdata is designed to layer on simply to existing JSON data, by adding a [`@class`](#class):
```
{
  "@class": "https://example.com/schema#Article",
  "title": "Introduction to Hyperdata"
}
```

Doing this specifies the following values via [implicit derivation](#implicit-derivation-of-namespaces-and-documents):
```
object class IRI:             https://example.com/schema#Article       (full-IRI)
namespace IRI:                https://example.com/schema#              (ns-IRI)
object property IRI:          https://example.com/schema#title         (full-IRI)
schema document IRI:          https://example.com/schema               (base-IRI)
short form name-fragments:    Article, title                           (name-fragment)
```
The expectation is that when you dereference the schema document `base-IRI` of `https://example.com/schema` (for example via HTTP GET), a successful response will be a Hyperdata document describing the schema within the established `https://example.com/schema#` namespace.

The dereferenced schema document is expected to specify:
- the `Article` class, described by an object with an [`@id`](#id) value of `https://example.com/schema#Article`, and
- the named properties of the `Article` class, such as `title`, which would be described by an object with an [`@id`](#id) value of `https://example.com/schema#title`.

In this manner, the `Article` class and its `title` property are universally and unambiguously named within the IRI space, allowing shared understanding of readily accessible Hyperdata.

By adding an [`@id`](#id) to the previous example, it can also be universally named within the IRI space, allowing it to be referenced and dereferenced within the web of Hyperdata.
```
{
  "@namespace": "https://example.com/schema#",
  "@class": "Article",
  "@id": "https://example.com/hyperdata-introduction#description",
  "title": "Introduction to Hyperdata"
}
```
Again, the expectation is that when you dereference the implicitly derived document `base-IRI` of `https://example.com/hyperdata-introduction`, a successful response will include the above example Hyperdata object. This example introduces all three JSON property extensions defined by Hyperdata: [`@namespace`](#namespace), [`@class`](#class), [`@id`](#id), and the short form name-fragment style.

## Mixin Property Modelling

Hyperdata is designed to follow the common single class based inheritance model of `class Person extends Agent`, however so as not to constrain its usage, object property names may be either a `name-fragment` as in previous examples, or a `full-IRI`. Usage of a `full-IRI` enables a mixin style composition of Hyperdata object descriptions.

```
{
  "@namespace": "https://example.com/schema#",
  "@class": "Article",
  "@id": "https://example.com/hyperdata-introduction#description",
  "title": "Introduction to Hyperdata",
  "https://example.com/version#sha256": "c61379b9379dbbea76341ac49e1ea5e85b9a441053a404f42d16cd2c2db81959"
}
```
In this example, the Hyperdata includes an object property name `https://example.com/version#sha256` from a different namespace (`https://example.com/version#`) to the default defined for the object (`https://example.com/schema#`).

## JSON Property Extensions

### @class
- **Value Space**: `full-IRI`
- **Lexical Space**: `name-fragment` OR `full-IRI`
- **Description**: Specifies the full-IRI of the class of this object.
- **Behavior**: 
  - If the value of [`@class`](#class) is a `full-IRI`, the value of [`@namespace`](#namespace) MUST be set to the computed `ns-IRI`.
    - Example: `"@class": "https://example.com/schema#Person` entails `"@namespace": "https://example.com/schema#"`
  - If the value of [`@class`](#class) is an `name-fragment`, the value MUST be appended to the in-scope [`@namespace`](#namespace) to compute it's `full-IRI` value.
    - Example: `{"@namespace": "https://example.com/schema#", "@class": "Person"}` entails a `full-IRI` of `https://example.com/schema#Person`
 
### @namespace
- **Value Space**: `ns-IRI`
- **Lexical Space**: `#` OR `ns-IRI`
- **Description**: Specifies the namespace of the current object and its children, unless they define a different namespace explicitly via [`@namespace`](#namespace) or implicitly via a `full-IRI` [`@class`](#class).
- **Behavior**:
  - If not defined on the current in-scope object, it MUST be inherited from the parent object.
  - If the value of [`@class`](#class) is a `full-IRI`, the `ns-IRI` value specified by this [`@namespace`](#namespace) MUST be ignored.

### Object Property Name Expansion
- **Value Space**: `full-IRI`
- **Lexical Space**: `name-fragment` OR `full-IRI`
- **Description**: Potentially establishes a unique universally quantified `full-IRI` for the property.
- **Behavior**:
  - If the value is an `name-fragment`, the value MUST be appened to the in-scope [`@namespace`](#namespace) to compute its potential `full-IRI` value.
    - Example: `{"@namespace": "https://example.com/schema#", "name": "Jon Doe"}` entails a `full-IRI` of `https://example.com/schema#name`
  - If the `full-IRI` cannot be determined to have a Hyperdata description after successfully dereferencing the associated Hyperdata document or some other out of band method, it should be treated as traditional JSON object property name (it's lexical string form `name-fragment`).
  - If the `full-IRI` belongs to a different Hyperdata namespace than the current in-scope [`@namespace`](#namespace), the property MAY be treated as a Mixin.
  - If the `full-IRI` belongs to a different [`@class`](#class) than the one specified for the in-scope object, the property MAY be treated as a Mixin.

### @id
- **Value Space**: `ns-IRI` OR `base-IRI` OR `full-IRI`
- **Lexical Space**: `ns-IRI` OR `base-IRI` OR `full-IRI` OR `#` OR (`#` `name-fragment`)
- **Description**: Establishes a unique universally quantified `IRI` for the object to which this property belongs, and which the object describes.
- **Behavior**:
  - If not defined on the current in-scope object, the object SHOULD be considered as existentially quantified.
  - If the lexical space is `#` OR (`#` `name-fragment`):
    - a `full-IRI` SHOULD be formed by appending it to a `base-IRI` established by common out of band methods, such as the IRI used in a successful HTTP GET dereference which provided the Hyperdata document.
    - The current in-scope namespace `ns-IRI` MUST NOT be considered when resolving it.
 

## IRI Name Concepts

### Hyperdata Document `base-IRI`

A Hyperdata Document is a JSON document that can be dereferenced using a unique identifier known as the `base-IRI`.

When accessed over HTTP, the document should be served with the media type `application/json`.
- `base-IRI` - a dereferencable IRI that does not include a `#name-fragment`.
  - `https://example.com/schema`
  
### Hyperdata Namespace `ns-IRI`

In a Hyperdata Document, the namespace it provides is identified by a `ns-IRI`. This defines a [`@namespace`](#namespace) within which Things, Properties, and Classes are named and described.
- `ns-IRI` - a namespace IRI which always ends with a `#`.
  - `https://example.com/schema#`

### Hyperdata Names `full-IRI`, `name-fragment`

In Hyperdata, Things, Properties, or Classes are given canonical `full-IRI` names.
- `full-IRI` - an IRI that ends with a `#name-fragment`.
  - `https://example.com/schema#Person`, `https://example.com/schema#name`

Within a Hyperdata document, a short form `name-fragment` can be used as a more concise way to denote a `full-IRI` when combined with a [`@namespace`](#namespace).
- `name-fragment` - a shorthand syntax used within Hyperdata documents to represent a `full-IRI`.
  - `{"@namespace": "https://example.com/schema#", "@class": "Person", "name": "Jon Doe"}`


### IRI ABNF Extensions
Hyperdata extends the [Internationalized Resource Identifiers (IRIs) - RFC3987](https://www.ietf.org/rfc/rfc3987.txt) ABNF with the following four terms, for use in Hyperdata.
```
   base-IRI       = scheme ":" ihier-part [ "?" iquery ]
   
   ns-IRI         = base-IRI "#"
   
   full-IRI       = ns-IRI "#" name-fragment
   
   name-fragment  = 1*( ipchar / "/" / "?" )
```

### Implicit Derivation of Namespaces and Documents

Hyperdata introduces a strict subset of IRI forms, which enable implicit derivation of both the namespace's `ns-IRI`, and the Hyperdata document's `base-IRI` from a single `full-IRI`.

Given a [`@class`](#class) value of `https://example.com/schema#Person`, we can implicitly determine the [`@namespace`](#namespace) to be `https://example.com/schema#` and the Hyperdata document's dereferenceable `base-IRI` to be `https://example.com/schema`.

Consequently, `{"@namespace": "https://example.com/schema#", "@class": "Person", "name": "Jon Doe"}` can succinctly be represented as `{"@class": "https://example.com/schema#Person", "name": "Jon Doe"}`.

To derive a `ns-IRI` from a `full-IRI`, remove everything after the `#`.
- `https://example.com/schema#Person` to `https://example.com/schema#`

To derive a `base-IRI` from a `full-IRI` or an `ns-IRI`, remove the `#` and any characters following it.
- `https://example.com/schema#Person` OR `https://example.com/schema#` to `https://example.com/schema`

To combine a `full-IRI` from a `ns-IRI` and a `name-fragment`, append the `name-fragment` to the `ns-IRI`.
- `https://example.com/schema#` + `Person` = `https://example.com/schema#Person`

