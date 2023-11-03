# Hyperdata

# notes
- schema can be # also
- need to establish a @base so # can be used, and name-fragment in @id

## Overview
This specification defines Hyperdata, an extension to JSON that adds IRI support, semantic annotations, and enables embedding schemas, identifiers, and classes in JSON objects. It is designed to enhance interoperability, data linking, and the self-descriptiveness of JSON data.

## IRI ABNF Extensions
```
   base-IRI       = scheme ":" ihier-part [ "?" iquery ]
   
   ns-IRI         = base-IRI "#"
   
   full-IRI       = ns-IRI "#" name-fragment
   
   name-fragment  = 1*( ipchar / "/" / "?" )
```
### Informally
- `base-IRI` - a dereferencable IRI that does not include a `#fragment`.
- `ns-IRI` - a namespace IRI that always end with a `#`.
- `full-IRI` - an IRI that end with a `#name-fragment`, identifiers used to refer to Things, Properties, and Types.
- `name-fragment` - a shorthand syntax used within hyperdata documents to denote a `full-IRI`.

### Converting
- To convert an `full-IRI` to a `ns-IRI` simply remove everything after the `#`.
- To convert an `full-IRI` or a `ns-IRI` to a `base-IRI` simply remove the `#` and any chars which follows it.
- To establish an `full-IRI` from an `ns-IRI` and a `name-fragment`, simply append the `name-fragment` to the `ns-IRI`.

## Properties

### @class
- **Type**: `name-fragment` || `full-IRI`
- **Description**: When the value is an `name-fragment` and concatenated with `@schema`, it creates an IRI identifying the class of the object.
- **Behavior**: 
  - If the value of `@class` is a `full-IRI`, the value of `@schema` MUST be set to the computed `ns-IRI`.
  - Example: `"@class": "https://example.org/Agents#Person` entails `"@schema": "https://example.org/Agents#"`
  - If the value of `@class` is an `name-fragment`, the value MUST be appened to the closest `@schema` to compute it's `full-IRI`
  - Example: `{"@schema": "https://example.org/Agents#", "@class": "Person"}` implies a `ns-IRI` of `https://example.org/Agents#Person`
 
### @schema
- **Type**: `ns-IRI` (IRI ending with `#`)
- **Description**: Specifies the schema of the current object and its children, providing semantic context and definitions.
- **Behavior**: 
  - If not present in the current object, it should be inherited from a parent object.

### @id
- **Type**: `name-fragment` || `full-IRI`
- **Description**: Serves as a unique identifier for the object of which this property belongs.
- **Usage**: Enables unique identification of objects within the JSON document or across different documents.

## Object Property Names
- **Type**: `name-fragment` || `full-IRI`
- **Usage**: Object properties should be concatenated to the value of `@vocab` to establish a `full-IRI` for the property, if the resulting value is defined in the schema then it must be considered as a canonical IRI identifying the property.
- Example: `{"@schema": "https://example.org/Agents#", "nick": "Jonny"}` implies a `full-IRI` of `https://example.org/Agents#nick`

## Constraints on JSON
- None

## Example 
```
[
  {
   "@schema": "https://example.org/Agents#",
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
