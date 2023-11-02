# Linked Objects: A JSON Extension Specification

## Overview
This specification defines an extension to the JSON format to include IRI support and semantic annotations, enabling the addition of schemas, identifiers, and types to JSON objects. It is designed to enhance interoperability, data linking, and the self-descriptiveness of JSON data.

## Properties

### @class
- **Type**: `ifragment` || `frag-IRI`
- **Description**: When the value is an `ifragment` and concatenated with `@schema`, it creates an IRI identifying the class of the object.
- **Behavior**: 
  - If the value of `@class` is a `frag-IRI`, the value of `@schema` MUST be set to the computed `hash-IRI`.
  - Example: `"@class": "https://example.org/Agents#Person` entails `"@schema": "https://example.org/Agents#"`
  - If the value of `@class` is an `ifragment`, the value MUST be appened to the closest `@schema` to compute it's `frag-IRI`
  - Example: `{"@schema": "https://example.org/Agents#", "@class": "Person"}` implies a `hash-IRI` of `https://example.org/Agents#Person`
 
### @schema
- **Type**: `hash-IRI` (IRI ending with `#`)
- **Description**: Specifies the schema of the current object and its children, providing semantic context and definitions.
- **Behavior**: 
  - If not present in the current object, it should be inherited from a parent object.

### @id
- **Type**: `ifragment` || `frag-IRI`
- **Description**: Serves as a unique identifier for the object of which this property belongs.
- **Usage**: Enables unique identification of objects within the JSON document or across different documents.

## Object Property Names
- **Type**: `ifragment` || `frag-IRI`
- **Usage**: Object properties should be concatenated to the value of `@vocab` to establish a `frag-IRI` for the property, if the resulting value is defined in the schema then it must be considered as a canonical IRI identifying the property.
- Example: `{"@schema": "https://example.org/Agents#", "nick": "Jonny"}` implies a `frag-IRI` of `https://example.org/Agents#nick`

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
