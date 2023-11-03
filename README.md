# Hyperdata

# notes
- schema can be # also
- need to establish a @base so # can be used, and hd-fragment in @id

## Overview
This specification defines Hyperdata, an extension to JSON that adds IRI support, semantic annotations, and enables embedding schemas, identifiers, and classes in JSON objects. It is designed to enhance interoperability, data linking, and the self-descriptiveness of JSON data.

## IRI ABNF Extensions
```
   hd-deref-IRI   = scheme ":" ihier-part [ "?" iquery ]
   
   hd-ns-IRI      = hd-deref-IRI "#"
   
   hd-IRI         = hd-ns-IRI "#" hd-fragment
   
   hd-fragment    = 1*( ipchar / "/" / "?" )
```
### Informally
- `hd-deref-IRI`, are dereferencable IRIs that never end with a `#fragment`.
- `hd-ns-IRI`, are IRIs that always end with an `#`, and can be considered name spaces.
- `hd-IRI`, are IRIs that end with a `#name`, and name Things, Properties, and Types.
- `hd-frag`, are identical to IRIs `ifragment`, but require at least 1 char, they never contain a `#`.

### Converting
- To convert an `hd-IRI` to a `hd-ns-IRI` simply remove everything after the `#`.
- To convert an `hd-IRI` or a `hd-ns-IRI` to a `hd-deref-IRI` simply remove the `#` and any chars which follows it.
- To establish an `hd-IRI` from an `hd-ns-IRI` and a `hd-fragment`, simply append the `hd-fragment` to the `hd-ns-IRI`.

## Properties

### @class
- **Type**: `hd-fragment` || `hd-IRI`
- **Description**: When the value is an `hd-fragment` and concatenated with `@schema`, it creates an IRI identifying the class of the object.
- **Behavior**: 
  - If the value of `@class` is a `hd-IRI`, the value of `@schema` MUST be set to the computed `hd-ns-IRI`.
  - Example: `"@class": "https://example.org/Agents#Person` entails `"@schema": "https://example.org/Agents#"`
  - If the value of `@class` is an `hd-fragment`, the value MUST be appened to the closest `@schema` to compute it's `hd-IRI`
  - Example: `{"@schema": "https://example.org/Agents#", "@class": "Person"}` implies a `hd-ns-IRI` of `https://example.org/Agents#Person`
 
### @schema
- **Type**: `hd-ns-IRI` (IRI ending with `#`)
- **Description**: Specifies the schema of the current object and its children, providing semantic context and definitions.
- **Behavior**: 
  - If not present in the current object, it should be inherited from a parent object.

### @id
- **Type**: `hd-fragment` || `hd-IRI`
- **Description**: Serves as a unique identifier for the object of which this property belongs.
- **Usage**: Enables unique identification of objects within the JSON document or across different documents.

## Object Property Names
- **Type**: `hd-fragment` || `hd-IRI`
- **Usage**: Object properties should be concatenated to the value of `@vocab` to establish a `hd-IRI` for the property, if the resulting value is defined in the schema then it must be considered as a canonical IRI identifying the property.
- Example: `{"@schema": "https://example.org/Agents#", "nick": "Jonny"}` implies a `hd-IRI` of `https://example.org/Agents#nick`

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
