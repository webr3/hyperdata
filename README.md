# Linked Objects: A JSON Semantic Extension Specification

## Overview
This specification defines an extension to the JSON format to include semantic annotations, enabling the addition of context, identifiers, and types to JSON objects. It is designed to enhance interoperability, data linking, and the self-descriptiveness of JSON data.

## Properties

### @schema
- **Type**: IRI (ending with `#`)
- **Description**: Specifies the schema of the current object and its children, providing semantic context and definitions.
- **Behavior**: 
  - If not present in the current object, it should be inherited from a parent object or determined based on the `@type` property.
  - If `@type` is a full IRI and `@schema` is not present, set `@schema` to the fragment-less IRI value of `@type` and append a `#`.
  - Example: `"@type": "https://example.org/Agents#Person` entails `"@schema": "https://example.org/Agents#"`

### @id
- **Type**: IRI
- **Description**: Serves as a unique identifier for the object of which this property belongs.
- **Usage**: Enables unique identification of objects within the JSON document or across different documents.

### @type
- **Type**: String || full IRI
- **Description**: When the value is a non-IRI string and concatenated with `@vocab`, it creates an IRI identifying the type of the object.
- **Behavior**: 
  - If `@type` is already a full IRI, `@vocab` is not required.
  - Provides the basis for setting `@schema` if it is not explicitly defined.

## Object Property Names
- Object properties should be concatenated to the value of `@vocab` to create IRIs, if the resulting value is defined in the schema then it must be considered as a canonical IRI identifying the property.
- Example: `{"@schema": "https://example.org/Agents#", "nick": "Jonny"}` implies a full IRI of `https://example.org/Agents#nick`

## Identifying IRIs in Values
- If a value contains a colon and conforms to the specifications of a full IRI, it should be treated as an IRI.

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
     "@type": "Person",
     "@id": "https://janedoe.example.org/#me",
     "name": "Jane Doe"
    }
  },
  ...
]
```
## Implications and Use Cases

- **Semantic Context**: Adds meaning and context to JSON objects, enhancing understandability.
- **Interoperability**: Facilitates integration with other systems and datasets.
- **Data Linking**: Enables creation of connected data graphs through linking and referencing.
- **Flexibility**: Supports both full and compacted IRIs for representation and linking.

## Challenges and Considerations
- **Dependency on External Schemas**: Relies on the availability and stability of external schemas.
- **Error Handling**: Requires clear guidelines and mechanisms for handling errors such as missing schemas or invalid IRIs.

## Linked-Objects-Schema
- TBD
