## GraphQL Types Overview

### Scalar Types

- **Definition**: Represents the basic types of data that GraphQL queries can return.
- **Built-in Scalar Types**:
  - `Int`: Integer value.
  - `Float`: Floating-point value.
  - `String`: UTF-8 character sequence.
  - `Boolean`: `true` or `false`.
  - `ID`: Unique identifier.

#### Example

```graphql
scalar DateTime
```

### Object Types

- **Definition**: Contains a set of fields that return specific types.
- **Usage**: Backbone of a GraphQL schema.

#### Example

```graphql
type Character {
  id: ID!
  name: String!
}
```

### List and Non-Null Modifiers

- **List**: Represents an array of items.
- **Non-Null**: Ensures the field cannot return `null`.

#### Example

```graphql
field: [String!]! # Non-null list of non-null strings
```

### Enums

- **Definition**: Represents a specific set of values.
- **Usage**: Reduces human error in queries.

#### Example

```graphql
enum Episode {
  NEWHOPE
  EMPIRE
  JEDI
}
```

### Interfaces

- **Definition**: Abstract type defining common fields shared by multiple object types.

#### Example

```graphql
interface Character {
  id: ID!
  name: String!
}

type Human implements Character {
  id: ID!
  name: String!
  homePlanet: String
}
```

### Unions

- **Definition**: Allows a field to return multiple object types.

#### Example

```graphql
union SearchResult = Human | Droid

search(text: "an") {
  ... on Human {
    name
    homePlanet
  }
  ... on Droid {
    name
    primaryFunction
  }
}
```

### Input Types

- **Definition**: Used as arguments in queries or mutations.

#### Example

```graphql
input ReviewInput {
  stars: Int!
  commentary: String
}

mutation {
  addReview(review: ReviewInput) {
    id
  }
}
```

### Directives

- **Definition**: Provide metadata or modify the schema or query behavior.
- **Built-in Directives**:
  - `@include(if: Boolean)`
  - `@skip(if: Boolean)`
  - `@deprecated(reason: String)`

#### Example

```graphql
type User {
  name: String @deprecated(reason: "Use 'fullName' instead.")
}
```

### Arguments

- **Definition**: Allow passing of variables into queries and mutations.
- **Default Values**: Optional arguments can have default values.

#### Example

```graphql
field(length: Int = 10): [String!]
```

### Schema

- **Definition**: Defines the entry points (query, mutation, subscription).

#### Example

```graphql
schema {
  query: Query
  mutation: Mutation
}

# Query Type
query {
  hero {
    name
  }
}
```

### Documentation in Schema

- **Single-line**: `" <description> "` or `# Comment`.
- **Multi-line**: `""" <description> """`.

#### Example

```graphql
"""
Represents a character in the Star Wars universe.
"""
type Character {
  id: ID!
  name: String!
}
```
