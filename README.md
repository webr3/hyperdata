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

### Deriving and Combining
- To derive a `ns-IRI` from a `full-IRI`, remove everything after the `#`.
  - `https://example.org/schema#Person` to `https://example.org/schema#`
- To derive a `base-IRI` from a `full-IRI` or an `ns-IRI`, remove the `#` and any characters following it.
  - `https://example.org/schema#Person` OR `https://example.org/schema#` to `https://example.org/schema`
- To combine a `full-IRI` from a `ns-IRI` and a `name-fragment`, simply append the `name-fragment` to the `ns-IRI`.
  - `https://example.org/schema#` + `Person` = `https://example.org/schema#Person`


### Example Document

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

## Properties

### @class
- **Type**: `name-fragment` OR `full-IRI`
- **Description**: When the value is an `name-fragment` and concatenated with `@namespace`, it creates an IRI identifying the class of the object.
- **Behavior**: 
  - If the value of `@class` is a `full-IRI`, the value of `@namespace` MUST be set to the computed `ns-IRI`.
  - Example: `"@class": "https://example.org/Agents#Person` entails `"@namespace": "https://example.org/Agents#"`
  - If the value of `@class` is an `name-fragment`, the value MUST be appened to the closest `@namespace` to compute it's `full-IRI`
  - Example: `{"@namespace": "https://example.org/Agents#", "@class": "Person"}` implies a `ns-IRI` of `https://example.org/Agents#Person`
 
### @namespace
- **Type**: `#` OR `ns-IRI`
- **Description**: Specifies the schema of the current object and its children, providing semantic context and definitions.
- **Behavior**: 
  - If not present in the current object, it should be inherited from a parent object.

### @id
- **Type**: `name-fragment` OR `full-IRI`
- **Description**: Serves as a unique identifier for the object of which this property belongs.
- **Usage**: Enables unique identification of objects within the JSON document or across different documents.

## Object Property Names
- **Type**: `name-fragment` || `full-IRI`
- **Usage**: Object properties should be concatenated to the value of `@vocab` to establish a `full-IRI` for the property, if the resulting value is defined in the schema then it must be considered as a canonical IRI identifying the property.
- Example: `{"@namespace": "https://example.org/Agents#", "nick": "Jonny"}` implies a `full-IRI` of `https://example.org/Agents#nick`

## Constraints on JSON
- None

## Example 
```
[
  {
   "@namespace": "https://example.org/Agents#",
   "@type": "Person",
   "@id": "https://jondoe.example.org/#me",
   "nick": "Jonny",
   "givenname": "Jon",
   "family_name": "Doe",
   "depiction": "https://jondoe.example.org/me.jpg",
   "homepage": "https://jondoe.example.org/",
   "interest": "https://dbpedia.org/resource/Film",
   "knows":  {
     "@id": "https://janedoe.example.org/#me",
     "name": "Jane Doe"
    }
  },
  ...
]
```
