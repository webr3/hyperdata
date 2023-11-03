# Hyperdata

This specification defines Hyperdata, an extension to JSON that adds IRI support, semantic annotations, and enables embedding schemas, identifiers, and classes in JSON objects. It is designed to enhance interoperability, data linking, and the self-descriptiveness of JSON data.

## Basic Concepts

### Hyperdata Document

A Hyperdata Document is a JSON document that can be dereferenced using a unique identifier known as the `base-IRI`.

When accessed over HTTP, the document should be served with the media type `application/json`.
- `base-IRI` - a dereferencable IRI that does not include a `#name-fragment`.
  - `https://example.org/schema`
  
### Hyperdata Namespace

In a Hyperdata Document, the namespace it provides is identified by a `ns-IRI`. This defines a `@namespace` within which Things, Properties, and Classes are named and described.
- `ns-IRI` - a namespace IRI which always ends with a `#`.
  - `https://example.org/schema#`

### Hyperdata Names

In Hyperdata, Things, Properties, or Classes are given canonical `full-IRI` names.
- `full-IRI` - an IRI that ends with a `#name-fragment`.
  - `https://example.org/schema#Person`, `https://example.org/schema#name`

Within a Hyperdata document, a short form `name-fragment` can be used as a more concise way to denote a `full-IRI` when combined with a `@namespace`.
- `name-fragment` - a shorthand syntax used within hyperdata documents to represent a `full-IRI`.
  - `{"@namespace": "https://example.org/schema#", "@class": "Person", "name": "Jon Doe"}`

### Implicit Derivation of Namespaces and Documents

Hyperdata introduces a strict subset of IRI forms, which enable implicit derivation of both the namespace's `ns-IRI`, and the hyperdata document's `base-IRI` from a single `full-IRI`.

Given a `@class` value of `https://example.org/schema#Person`, we can implicitly determine the `@namespace` to be `https://example.org/schema#` and the hyperdata document's dereferenceable `base-IRI` to be `https://example.org/schema`.

Consequently, `{"@namespace": "https://example.org/schema#", "@class": "Person", "name": "Jon Doe"}` can succinctly be represented as `{"@class": "https://example.org/schema#Person", "name": "Jon Doe"}`.

To derive a `ns-IRI` from a `full-IRI`, remove everything after the `#`.
- `https://example.org/schema#Person` to `https://example.org/schema#`

To derive a `base-IRI` from a `full-IRI` or an `ns-IRI`, remove the `#` and any characters following it.
- `https://example.org/schema#Person` OR `https://example.org/schema#` to `https://example.org/schema`

To combine a `full-IRI` from a `ns-IRI` and a `name-fragment`, append the `name-fragment` to the `ns-IRI`.
- `https://example.org/schema#` + `Person` = `https://example.org/schema#Person`

## JSON Property Extensions

### @class
- **Value Space**: `full-IRI`
- **Lexical Space**: `name-fragment` OR `full-IRI`
- **Description**: Specifies the full-IRI of the class of this object.
- **Behavior**: 
  - If the value of `@class` is a `full-IRI`, the value of `@namespace` MUST be set to the computed `ns-IRI`.
    - Example: `"@class": "https://example.org/schema#Person` entails `"@namespace": "https://example.org/schema#"`
  - If the value of `@class` is an `name-fragment`, the value MUST be appened to the in-scope `@namespace` to compute it's `full-IRI` value.
    - Example: `{"@namespace": "https://example.org/schema#", "@class": "Person"}` entails a `full-IRI` of `https://example.org/schema#Person`
 
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
   "@namespace": "https://example.org/schema#",
   "@class": "Person",
   "@id": "https://jondoe.example.org/#me",
   "name": "Jon Doe",
   "homepage": {
     "href": "https://jondoe.example.org/",
     "title": "Jon Doe's Homepage"
   },
   "knows":  {
     "@id": "https://janedoe.example.org/#me",
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

