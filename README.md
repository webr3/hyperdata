# Hyperdata

# notes
- schema can be # also
- need to establish a @base so # can be used, and lo-ifragment in @id

## Overview
This specification defines Hyperdata, an extensions to JSON that adds IRI support, semantic annotations, and enables embedding schemas, identifiers, and classes in JSON object. It is designed to enhance interoperability, data linking, and the self-descriptiveness of JSON data.

## IRI ABNF Extensions
```
   lo-deref-IRI   = scheme ":" ihier-part [ "?" iquery ]
   
   lo-ns-IRI      = lo-deref-IRI "#"
   
   lo-IRI         = lo-ns-IRI "#" lo-ifragment
   
   lo-ifragment   = 1*( ipchar / "/" / "?" )
```
### Informally
- `lo-deref-IRI`, are dereferencable IRIs that never end with a `#fragment`.
- `lo-ns-IRI`, are IRIs that always end with an `#`, and can be considered name spaces.
- `lo-IRI`, are IRIs that end with a `#name`, and name Things, Properties, and Types.
- `lo-frag`, are identical to IRIs `ifragment`, but require at least 1 char, they never contain a `#`.

### Converting
- To convert an `lo-IRI` to a `lo-ns-IRI` simply remove everything after the `#`.
- To convert an `lo-IRI` or a `lo-ns-IRI` to a `lo-deref-IRI` simply remove the `#` and any chars which follows it.
- To establish an `lo-IRI` from an `lo-ns-IRI` and a `lo-ifragment`, simply append the `lo-ifragment` to the `lo-ns-IRI`.

## Properties

### @class
- **Type**: `lo-ifragment` || `lo-IRI`
- **Description**: When the value is an `lo-ifragment` and concatenated with `@schema`, it creates an IRI identifying the class of the object.
- **Behavior**: 
  - If the value of `@class` is a `lo-IRI`, the value of `@schema` MUST be set to the computed `lo-ns-IRI`.
  - Example: `"@class": "https://example.org/Agents#Person` entails `"@schema": "https://example.org/Agents#"`
  - If the value of `@class` is an `lo-ifragment`, the value MUST be appened to the closest `@schema` to compute it's `lo-IRI`
  - Example: `{"@schema": "https://example.org/Agents#", "@class": "Person"}` implies a `lo-ns-IRI` of `https://example.org/Agents#Person`
 
### @schema
- **Type**: `lo-ns-IRI` (IRI ending with `#`)
- **Description**: Specifies the schema of the current object and its children, providing semantic context and definitions.
- **Behavior**: 
  - If not present in the current object, it should be inherited from a parent object.

### @id
- **Type**: `lo-ifragment` || `lo-IRI`
- **Description**: Serves as a unique identifier for the object of which this property belongs.
- **Usage**: Enables unique identification of objects within the JSON document or across different documents.

## Object Property Names
- **Type**: `lo-ifragment` || `lo-IRI`
- **Usage**: Object properties should be concatenated to the value of `@vocab` to establish a `lo-IRI` for the property, if the resulting value is defined in the schema then it must be considered as a canonical IRI identifying the property.
- Example: `{"@schema": "https://example.org/Agents#", "nick": "Jonny"}` implies a `lo-IRI` of `https://example.org/Agents#nick`

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
