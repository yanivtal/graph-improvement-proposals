---
GRC: "20"
Title: Knowledge Graph
Authors: Yaniv Tal, Byron Guina
Created: 2024-11-21
Stage: Draft
Version: 0.1.0
Discussions-To: https://forum.thegraph.com/t/grc-20-knowledge-graph/6161
---

# GRC-20: Knowledge Graph

## Abstract

This GRC introduces a standard for storing and representing knowledge on The Graph. It can be used by any application that wants to create and consume interoperable information across applications.

Data is defined as raw bits that are stored and transmitted. Information is data that is decoded and interpreted into a useful structure. Knowledge is created when information is linked and labeled to attain a higher level of understanding. This document outlines the valid serialization format for the knowledge data that is anchored onchain, shared peer-to-peer or stored locally. Using this standard, any application can access the entire knowledge graph and produce knowledge that can become part of The Graph.

## Motivation

Knowledge graphs are the most flexible way of representing information. Knowledge is highly interconnected across applications and domains, so flexibility is important to ensure that different groups and individuals can independently extend and consume knowledge in an interoperable format. Most apps define their own schemas. Those schemas work for a specific application but getting apps to coordinate on shared schemas is difficult. Once data is produced by an app, custom code has to be written to integrate with each app. Migrations, versioning and interpreting data from different apps becomes even more difficult as schemas evolve. These are some of the reasons that instant composability and interoperability across apps is still a mostly unsolved problem. The original architects of the web understood this and tried to build the semantic web which was called Web 3.0 twenty years ago. Blockchains and decentralized networks give us the tools we need to build open and verifiable information systems. Solving the remaining composability challenges will enable an interoperable web3.

Computer scientists widely understand triples to be the best unit for sharing knowledge across organizational boundries. RDF is the W3C standard for triples data. A new standard is necessary for web3 for several reasons: IDs in RDF are URIs which typically point to servers that are controlled by corporations or individuals using https. This breaks the web3 requirement of not having to depend on specific server operators. Additionally, RDF doesn't support the property graph model, which is needed to describe facts about relationships between entities. Some of RDF's concepts are cumbersome and complex which has hindered its adoption beyond academic and niche enterprise settings. For these reasons, a new standard is being proposed that is web3 native, benefits from the latest advancements in graph databases and can be easily picked up by anyone who wants to build the decentralized web.

## Specification

Knowledge graphs are comprised of entities that represent people, places, things, concepts, or anything else. These entities have relations that link them into a graph. Knowledge is published onto The Graph using onchain transactions. Data can be stored offchain on a content addressed storage network and then anchored onchain. Each space for a community or individual has its own knowledge graph and its own governance or access controls. Any entity can reference any other entity across the entire graph, creating a global unified graph. Knowledge graph data doesn't have to be onchain, it can also be transmitted peer-to-peer between users and stored locally.

### 1. Encoding

Data is encoded using Protobufs. Protocol Buffers are a language-neutral, platform-neutral extensible mechanism by Google for serializing structured data. It's chosen for speed, compactness and broad language support.

### 2. IDs

All IDs must be globally unique UUIDs. They should be created using the UUID4 standard and byte encoded over the wire. When string encoding in clients, the UUIDs should have their dashes stripped out. If an entity is coming from a system that already has guaranteed globally unique IDs of arbitrary length, they can be deterministically converted into valid ID by taking an md5 hash of the string, seeding that into a UUID4 generator.

Example ID: `2a5fe010-078b-4ad6-8a61-7272f33729f4`

### 3. Spaces

A space is a logical grouping that represents a group or individual, a knowledge graph and a governance process or access control system for updating information or making changes to the space. A space can be a personal space or a public space. Personal spaces can be private and run local first on a user's device, be shared with a group of people or be published onto a public blockchain for global consistency. Public spaces must be onchain with a public governance system.

### 4. Entities

Knowledge on The Graph is composed of Entities and Relations. Entities represent people, places, things, concepts, or anything else. Entities are themselves unstructured and can contain arbitrary data. The first time an entity is created, the client is responsible for generating a valid globally unique ID. Enforcement of global ID uniqueness is expected of each space's governance.

Data points can be added about an entity by adding properties (key/value pairs). Different spaces can have different points of view on the same (Entity, Property) pair. For example, a space for Americans can assert that the Name of an entity is "Fries" and a British space might call the same entity "Chips" - but they would both reference the same entity ID.

If there are multiple values for a single (Space, Entity, Property) combination, subsequent values get treated as an upsert, replacing the previous value for that tuple. This applies to the native types and doesn't include relations which support many to many relationships.

```proto
message Entity {
  bytes id = 1;
  map<bytes, Value> properties = 2;
}
```

#### 4.3.1 Value

The value is an object that includes the value along with optional metadata. Metadata such as the type and formatting should be set on the Property itself. Options that are included with the value take precedence over the same options set on the Property entity.

| Name            | Value | Description                                               |
| --------------- |-------| --------------------------------------------------------- |
| Value           | 1     | The encoding for the value depends on the value type      |
| Options         | 2     | Value type specific options                               |

The corresponding protobuf message is:

```proto
message Value {
  string value = 1;
  Options options = 2;
}
```

Options can be included with a value and is specific to the value type. Additional options can be added in future spec versions.

```proto
message Options {
  bytes unit = 1;
  string format = 2;
}
```

#### Example Entity in JSON

```jsonc
{
  "id": "2a5fe010-078b-4ad6-8a61-7272f33729f4",
  "properties": {
      "6ab0887f-b6c1-4628-9eee-2234a247c1d2": {
        "value": "San Francisco"
      }
  }
}
```


### 7. Properties

Entities have properties which are themselves defined as entities. A property describes an attribute or a field of an entity. For example, a property might describe a Name, Description, Birthday or any other attribute of the entity. The property may or may not be included as a property on one of the entity's types. If a value type is not included when creating a property, it defaults to being a Text property.

| Name            | Type | ID                     |
| --------------- |------|----------------------- |
| Property        | Type | 2f92f4cc-0d97-4b7f-a039-44b61a3c3260 |

The Property type has the following attributes:

| Name           | Type            | Description                                                          |
| -------------- |-----------------| -------------------------------------------------------------------- |
| Value type     | Relation\<Type> | The entity ID for the type a value should have.                      |
| Relation value types | Relation\<Type> | When the property is for a relation, hints for the possible types on the relation entity |
| Unit           | Relation\<Unit> | An optional unit                                                                               |
| Format         | Text            | An optional format string                                                                      |


Attributes are used for native types like Text, Number and Time. The Value Types should reference the entity IDs created for the native types.

| Name                  | ID                                   |
| ----------------------| ------------------------------------ |
| Value type            | 9fc412d5-d53c-45c9-9e1f-2eee32b971f8 |
| Relation value types  | 33f2784f-4743-4c70-821a-cfd49adb8aee |
| Unit                  | 386d9dde-4c10-4099-af42-0e946bf2f199 |
| Format                | c5d181a2-6593-4c2c-b0da-8396bab2d8fb |


There are 6 native value types. A princple used for naming the native types is to prioritize accessibility and familiarity for non-developers. There will be many different types of applications producing and consuming knowledge and the more consistent, familiar and accessible producing and consuming raw knowledge is, the more people will be able to participate in the knowledge economy.

| Name         | ID                     | Description                                               |
| -------------| ---------------------- | --------------------------------------------------------- |
| Text         | 992321fd-c288-4dac-bf89-27f52e448557 | A string of characters                                    |
| Number       | eb3155c5-7e70-406f-ae70-17f029a6a76d | A decimal value                                           |
| Checkbox     | c6345346-2e40-456f-a7d8-ffdbdd81fec7 | Can either be `true` or `false`                           |
| URL          | bb8c1304-7f3a-43e7-a444-d3aa589fb096 | A URI to a resource with a protocol identifier            |
| Time         | 3c62afbd-c430-438a-83b0-61c60de5f03f | A date and time or interval                               |
| Point        | bc76a151-fa46-4a9a-9590-c40e1738f452 | For geo locations or other coordinates                    |


In addition, properties can have Value Type: Relation, which isn't a native type, but the allows us to define relation types as part of the schema. 

| Name         | Type      | ID                     |
| ------------ |-----------|----------------------- |
| Relation     | Type      | 1dc83461-cb6c-4af7-bca1-84cb9554c84a |

For each native value type, if a value is provided that is invalid, the triple should be dropped and treated as unset. Null cannot be provided as a value. Each native value type is UTF-8 encoded as a string.

### 8. Relations

Relations describe the edges of a graph. For example a Company can have Team Members. Each Team Member relation can have a property describing when the person joined the team. This is a model that is commonly called a property graph. Relations are enocded in 2 parts: a lightweight relation and an optional relation entity with details about the relationship. 

Relations are ordered. They can be reordered independently without conflicts using Fractional Indexing with an arbitrary level of precision between the indices of the items the user wants to move the item between. A fractional index must include randomness to allow items to be inserted between items with minimal chance of collision. This way moving the order of an item doesn't require changing the index of any of the other items.

A relation is a native object type with a lightweight representation and a reference to an entity for adding additional properties.

| Name          | Type | Description                                                                   |
| ------------- |------| ----------------------------------------------------------------------------- |
| ID            | ID   | The ID for this relation                                                      |
| Type          | ID   | The ID of the relation type                                                   |
| From entity   | ID   | The entity ID this relation is from                                             |
| From property | ID   | An optional property ID for creating a relation from a specific property value  |
| To entity     | ID   | The entity ID this relation is pointing to                                      |
| To property   | ID   | The entity ID this relation is pointing to                                      |
| To space      | ID   | An optional space ID to link to an entity specifically in a given space         |
| Verified      | Checkbox | Whether or not the 'to space' is known by the source space to represent the 'to entity' |
| Entity        | ID   | A new entity ID that can be used to add properties and types to the relation    |
| Index         | Text | An alphanumeric fractional index describing the relative position of this item |


ID, Type, From entity, To entity, and Entity are required. Linked entities don't have to be created before they're referenced by relations. A globally unique ID must be created for the Entity though no additional information has to be provided for the relation entity. This way there is already an ID available for adding data about the relation.

The From entity, To entity, Entity, and Type fields of the lightweight relations are immutable. To space, Verified, and Index can be updated. The relation entity can be updated to add/remove additional properties or add/remove additional types.

#### 8.1 Relation Types

A Relation Type is both a property and a type, but specifically for defining one to one, one to many, and many to many relations. A Relation Type can have a name like `Team member`, `City` or a more graph style name like `Born in`. A relation must have exactly one Relation Type.

| Name          | Type      | ID                     |
| ------------- |-----------|----------------------- |
| Relation Type | Type      | 0509279a-713d-4667-a412-24fe3757f554 |

A Relation Type is also of type `Type` and has the following attributes:

| Name                     | Type              | Description                                                                       |
| ------------------------ |-------------------| --------------------------------------------------------------------------------- |
| Relation value types     | Relation \<Type>  | A list of types that the relation To entity may have. Used as a hint              |
| One max                  | Checkbox          | Whether to limit the number of relations from a given entity to no more than 1    |
| Cascades                 | Checkbox          | Whether to move and delete the to entity and relation entity with the from entity |

| Name                     | Type          | ID                     |
| ------------------------ |---------------|----------------------- |
| Relation value types     | Relation\<Type> | 3b2dca52-f1bf-45c0-ab1c-b4f883cf51d1 |
| One max                  | Checkbox      | dfeb3c65-28f5-4962-9d66-6df94e2f9bb6 |
| Cascades                 | Checkbox      | 69a2489b-1e2e-42d9-93f3-0779e44ebca6 |


### 6. Types

Types describe the structure of information. In an interconnected graph that can be used by any application, types should be helpful but not get in the way. Applications should be able to evolve independently without breaking others. The primary goal is composition and reuse. There should be a draw towards reuse and sharing, while empowering people to customize whatever they want.

Every entity has an implicit Types relation and can have zero, one or many types.

| Name      | Type | ID                     |
| --------- |------|----------------------- |
| Type      | Type | 00c3d9d2-78a9-4b92-9772-bbdc57a28614 |

Types may have zero, one or many properties and relation types defined on them. The Broader types are used to define a subtype hierarchy, with the subtypes pointing to their broader types. Subtypes inherit properties from their broader types.

| Name           | Type                      | ID                     |
| -------------- |---------------------------|----------------------- |
| Properties     | Relation \<Property>      | a69ffb85-e3aa-4c1d-92c1-e33a2480ab80 |
| Broader types  | Relation \<Type>          | 14df474e-4f48-4117-9619-d99e4b3afd3e |



### 5. Native value types

There are 6 built in native value types defined. For each native value type, if a value is provided that is invalid, the triple should be dropped and treated as unset. Null cannot be provided as a value. Each native value type is UTF-8 encoded as a string.

#### 5.1 Text

An arbitrary length string of characters. Text may include newline characters or markdown. Clients may choose which attributes to support markdown for. Example: "Radiohead"

Options: `language`

A language option can be provided to specify a language for the text. The language must be an entity ID for a Language entity. If no language is specified and the text is linguistic, it is assumed to be in US English.

#### 5.2 Number

Numbers are encoded as strings. One decimal point may be used. An optional negative sign (-) can be included at the beginning for negative values. No other symbols such as commas may be used. Additional metadata for units and formatting can be included as options.

Examples:
* "1234.56"
* "-789.01"

Options: `format`, `unit`

The format option uses the ICU decimal format. For example:
`¤#,##0.00` would render a value like `$1,234.56`.

The unit option can be included to define a specific currency or other unit to use. The value must be the entity ID of a Currency or Unit entity.

Additional attributes can be added to Number attributes to specify hints for database optimizations. This can allow application developers to add additional constraints that can be useful for choosing more efficient storage types.

#### 5.3 Checkbox

A checkbox can either be `true` or `false`. It's a boolean field named for normal people. The string encoding of the checkbox field must be "1" for `true` and "0" for `false`. Checkbox values can be rendered with alternate UI components such as switches and toggles.

#### 5.4 URL

A URL value type is technically a URI that starts with the protocol identifier followed by a :// and the resource. The initial supported protocols are: graph, ipfs, ar, https.

Examples:
* "ipfs://QmVGF2e9WqF8g39W81eCQxecz7Bdi5o2DFSJ5BQxwfveYB"
* "ht<span>tps://</span>geobrowser.io"

#### 5.5 Time

Time is represented as an ISO-8601 format string. A time value can be used to describe both date and time as well as durations. Clients should include timezones when creating times. In rare exceptions, if the timezone is omitted, the time is interpreted as being in the consumer's local time. Example: `2024-11-03T11:15:45.000Z`

Options: `format`

A format string using the Unicode Technical Standard #35.

Examples:
* "MMMM do, yyyy" - July 4th, 2024
* "M/d/yyyy" - 7/4/2024
* "h:mma" - 4:45pm
* "EEEE h:mma" - Friday 5:00pm

#### 5.6 Point

A point is a location specified with x and y coordinates string encoded. Points can be used for geo locations or other cartesian coordinate systems. Clients may choose how many dimensions to support. Example: `"12.554564, 11.323474"`

The value must have valid decimal numbers separated by a comma. A space after the comma is optional. For geo locations, the latitude and longitude must be provided with latitude first. Positive/negative signs indicate north/south for latitude and east/west for longitude respectively.


### 9. Implicit entity properties

All entities implicitly have the Entity type which contains the following properties:

| Name            | Type               | Description                                               |
| --------------- |--------------------| --------------------------------------------------------- |
| Name            | Text               | The name that shows up wherever this entity is referenced |
| Types           | Relation \<Type>   | The one or many types that this entity has                |
| Description     | Text               | A short description that is often shown in previews       |
| Cover           | Image              | A banner style image that is wide and conveys the entity  |
| Blocks          | Relation \<Block>  | The blocks of rich content associated with this entity    |

The ideal cover image dimensions is 2384x640px though other dimensions can be used.

| Name        | ID                     |
| ------------| ---------------------- |
| Name        | 6ab0887f-b6c1-4628-9eee-2234a247c1d2 |
| Types       | ebbb7452-0465-4ad9-8c70-6e2ddd96178b |
| Description | f0e78bb5-00e9-4ec7-a0b0-32b01b452e1f |
| Cover       | 8b98dab6-0811-4c36-b8a5-fd3f89729fe3 |
| Blocks      | 29785ffb-5ce6-4a9b-b0dc-b138328ba334 |


### 10. System properties

System derived properties are not serialized but they are part of the standard for data that is computed by indexers and made available on entities. Clients should show these fields as non-editable.

| Name           | Type                | Description                                                                             |
| -------------- |---------------------| --------------------------------------------------------------------------------------- |
| Created at     | Time                | The time this entity was first seen by the indexer                                      |
| Updated at     | Time                | The most recent type this entity was updated by the indexer                             |
| Created by     | Text                | The blockchain address of the account that signed the transaction to create this entity |
| Versions       | Relation \<Version> | A reference to previous versions of this entity                                         |


### 11. Space type

A space is also a type but public and personal spaces are special because they also include the deployment of a smart contract for governance and access controls. Any entity can also be a space. Once enough interest and specialization forms around an entity, it can be converted into its own space with its own governance.

| Name      | Type | ID                     |
| --------- |------|----------------------- |
| Space     | Type | 8029f4f7-6691-4011-a60c-9685728a9b2f |

A space has the following properties:

| Name            | Type               | Description                                                        |
| --------------- |--------------------| ------------------------------------------------------------------ |
| Broader spaces  | Relation \<Space>  | The spaces that this is a subspace of                              |
| Subspaces       | Relation \<Space>  | Spaces that are more granular for drilling down                    |

The Broader [thing] and Sub[thing] is a convention for relations to denote drilling down or up when something is hierarchical, chosen for natural language. The Broader spaces and Subspaces relations are system properties that can be queried as standard properties but are derived from the space contracts.

### 12. Entity content types

A separate Content spec will be provided with a standard set of content types. Two types are specified here since they are referenced in the _Implicit entity properties_ section.

#### 12.1 Image

Images are defined as a type with properties that can be used for provenance, attribution and other metadata. This way images can be reused in many places with proper attribution.

| Name           | Type                          | Description                                      |
| -------------- |-------------------------------| ------------------------------------------------ |
| Width          | Number                        | The width in pixels                              |
| Height         | Number                        | The height in pixels                             |

| Name       | Type | ID                     |
| ---------- |------|----------------------- |
| Image      | Type | 772172cb-7abf-422e-8b8b-10ef17738402 |

#### 12.2 Block

Every entity can have blocks comprising the content for a page. Apps can choose to support different block types. The standard set of block types will be published in the Content spec.

| Name       | Type          | ID                     |
| ---------- |---------------|----------------------- |
| Block      | Relation Type | 8548f78a-2b37-477d-b2e2-924c3373ee12 |

### 13. Ops

An op represents a single atomic operation that produces or modifies knowledge. Ops are provided as an ordered list and must be processed in sequence.

```proto
message Op {
  OpType type = 1;
  Entity entity = 2;
  Relation relation = 3;
  Property property = 4;
}
``` 

```proto
message Relation {
  bytes id = 1;
  bytes type = 2;
  bytes from_entity = 3;
  bytes from_property = 4;
  bytes to_entity = 5;
  bytes to_property = 6;
  bytes to_space = 7;
  bytes entity = 8;
  bool verified = 9;
  string index = 10;
}
```

```proto
message Property {
  bytes id = 1;
  bytes type = 2;
  string name = 3;
}
``` 


```proto
enum OpType {
  CREATE_ENTITY = 1;
  UPDATE_ENTITY = 2;
  DELETE_ENTITY = 3;
  CREATE_RELATION = 4;
  UPDATE_RELATION = 5;
  DELETE_RELATION = 6;
  UNSET_PROPERTIES = 7;
  CREATE_PROPERTY = 8;
  ARCHIVE_PROPERTY = 9;
  MOVE_ENTITY = 10;
  MERGE_ENTITIES = 11;
  BRANCH_ENTITY = 12;
}
```

#### 13.1 Create entity

When the OpType is `Create entity`, the entity field must be included and must include an ID.

#### 13.4 Update entity

When the OpType is `Update entity`, just the entity field should be included with the entity ID to update along with a set of properties to add or upsert. To unset a property value, use the `Unset properties` op.

#### 13.4 Delete entity

When the OpType is `Delete entity`, just the entity field should be included with the entity ID. Indexers should delete all properties for deleted entities in the space.

#### 13.5 Create relation

When creating a relation, the `Create relation` op type should be used instead of `Create entity`. The relation field must be included with `ID`, `Type`, `From entity`, and `To entity` all required.

#### 13.6 Delete relation

When the OpType is `Delete relation`, just the relation field should be included with the relation ID. Indexers should delete all fields on the delation but keep the relation entity.

QUESTION: Should there be a way to delete the relation entity without a separate op?

#### 13.6 Reorder relation

When the OpType is `Reorder relation`, just the relation field should be included with the relation ID and the new index.

#### 13.6 Unset properties

When the OpType is `Unset property`, just the entity field should be included with the entity ID and the properties map with the property IDs to unset as the keys and the empty string as the values.

#### 13.6 Create property

When the OpType is `Create property`, just the property field should be included with id, type, and name included.

#### 13.6 Archive property

When the OpType is `Archive property`, just the property field should be included with the ID included. This is used to indicate that a property is meant to be deprecated. Data may still exist for this property.


### 14. Edits

Changes are published to the knowledge graph through `edits`. An edit can include any number of changes to any entity in a space.

| Name            | Type            | Value | Description                                               |
| --------------- |-----------------|-------| --------------------------------------------------------- |
| Version         | string          | 1     | A semver version string for the knowledge serialization   |
| Type            | ActionType      | 2     | The type of action this edit is encoding                  |
| ID              | string          | 3     | A valid ID representing this edit                         |
| Name            | string          | 4     | The name of the edit (short commit message)               |
| Ops             | Op[]            | 5     | A list of triple operations to perform                    | 
| Authors         | string[]        | 6     | The list of authors who worked on this edit               |
| Language        | bytes           | 7     | The language for all text fields in this edit (default Eng) |

The corresponding protobuf message is:

```proto
message Edit {
  string version = 1;
  ActionType type = 2;
  string id = 3;
  string name = 4;
  repeated Op ops = 5;
  repeated string authors = 6;
}
```

#### Example Edit in JSON

```jsonc
{
  "version": "1.0.0",
  "type": 1, // Add edit
  "id": "JVrauVCjqsuKqArK3dutYb",
  "name": "Add a new city",
  "ops": [
    {
      "type": 1, // Create triple
      "triple": {
        "entity": "Gw9uTVTnJdhtczyuzBkL3X",
        "attribute": "7UiGr3qnjZfRuKs3F3CX61",
        "value": {
            "type": 1, // Text
            "value": "San Francisco"
        }
      }
    }
  ],
  "authors": ["7UiGr3qnjZfRuKs3F3CX61"]
}
```

### 15. Imports

Spaces are abstracted from the underlying infrastructure. A space contract can be deployed on one L2 and then move to another L2. An import file includes all of the data from a previous space and can be used to bootstrap the space in the new environment.

```proto
message Import {
  string version = 1;
  ActionType type = 2;
  string previousNetwork = 3;
  string previousContractAddress = 4;
  repeated string edits = 5;
}
```

#### Example Import in JSON

```json
{
  "version": "1.0.0",
  "type": "import",
  "previousNetwork": "7UiGr3qnjZfRuKs3F3CX61",
  "previousContract": "0x1A39E2Fe299Ef8f855ce43abF7AC85D6e69E05F5",
  "edits": [
    "ipfs://QmVGF2e9WqF8g39W81eCQxecz7Bdi5o2DFSJ5BQxwfveYB"
  ]
}
```

### 16. Action Types

Actions are logical functions that can be performed on a space in different environments. Public spaces should secure these actions behind a public governance system, personal spaces can secure these actions behind an access control system, and private spaces can run these actions locally. Actions can be used to describe a log of user intents as well as a log of a space's accepted actions.

```proto
enum ActionType {
  ADD_EDIT = 1;
  ADD_SUBSPACE = 2;
  REMOVE_SUBSPACE = 3;
  IMPORT_SPACE = 4;
  ARCHIVE_SPACE = 5;
}
```

#### 16.1 Add edit

Params: `edit` - The URI to the edit protobuf file. Edit files must be hosted on decentralized storage.

#### 16.2 Add subspace

Params: `subspace` - The space entity ID to add as a subpace of the current space.

#### 16.3 Remove subspace

Params: `subspace` - The space entity ID to remove as a subpace of the current space.

#### 16.4 Import space

Params: `import` - The URI to the import protobuf file. Import files must be hosted on decentralized storage.

#### 16.5 Archive space

Flags the space as unused. Indexers and clients can still index the space if desired but an archived space is no longer maintained. Public spaces whose data was stored on decentralized storage and anchored onchain should continue to be available forever to anyone who wants to use or reference them.

## Copyright waiver

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).

