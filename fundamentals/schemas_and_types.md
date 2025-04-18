# Schemas and Types

## Overview

- Depending on libraries, types/fields/resolver functions might come built-in or allow you to define types/fields more ergonomically via schema definition language (SDL). You can then write resolver functions for the corresponding fields separately.

```graphql
type <object_type> {
    <field_1>: String! # non-null type
    <field_2>: [Episode!]!
}
```

- `String` is one of the built-in scalar types.
- Scalar types cannot have sub-selections; they are "leaves" or the "end" of the query.
  - **Examples**: `Int`, `Float`, `String`, `Boolean`, `ID`.
- The exclamation mark (`!`) guarantees non-null values or arrays with at least one element.

- Schemas must support query, mutation, and subscription operations.

```graphql
schema {
  query: MyQueryType
  mutation: MyMutationType
  subscription: MySubscriptionType
}
```

- Custom scalar types can be created in most GraphQL service implementations.

```graphql
scalar Date
```

- It is up to the implementation to define how the type should be serialized, deserialized, and validated.
  - **Example**: Keeping a "Date" type as an integer timestamp.

---

## Enum Types

- Enum types define a set of expected outputs, reducing human error.

```graphql
enum Episode {
  NEWHOPE
  EMPIRE
  JEDI
}
```

---

## Type Modifiers

- By default, types are nullable and singular in GraphQL.

### Non-Null Types

- Use the `!` symbol to mark fields as non-nullable.

```graphql
type Character {
  name: String!
}

{
  droid(id: null) {
    name
  }
}
```

### List Types

- Lists are used to define arrays.

```graphql
myField: [String]      # Example 1
myField: [String!]     # Example 2
myField: [String!]!    # Example 3
```

#### Validation Scenarios

```graphql
myField: [String]      # Valid: null, [], ['a'], ['a', null, 'b']
myField: [String!]     # Valid: [], ['a'], ['a', 'b'], ['a', null, 'b']
myField: [String!]!    # Valid: [], ['a'], ['a', 'b']; Invalid: null, ['a', null, 'b']
```

---

## Abstract Types

### Interface Type

- Interfaces are useful for returning objects or sets of objects that might be of several different types.

```graphql
interface Character {
  id: ID!
  name: String!
}

type Human implements Character {
  id: ID!
  name: String!
  starships: [Starship]
  totalCredits: Int
}

type Droid implements Character {
  id: ID!
  name: String!
  primaryFunction: String
}
```

### Inline Fragments

- Used to query fields on specific object types.

```graphql
  hero(episode: JEDI) {
    name
    ... on Droid {
      primaryFunction
    }
  }
```

### Union Types

- Union types group multiple types without shared fields.

#### Query

```graphql
    union SearchResult = Human | Droid | Starship

    search(text: "an") {
      __typename
      ... on Human {
        name
        height
      }
      ... on Droid {
        name
        primaryFunction
      }
      ... on Starship {
        name
        length
      }
    }
```

#### Response

```json
{
  "data": {
    "search": [
      {
        "__typename": "Human",
        "name": "Han Solo",
        "height": 1.8
      },
      {
        "__typename": "Human",
        "name": "Leia Organa",
        "height": 1.5
      },
      {
        "__typename": "Starship",
        "name": "TIE Advanced x1",
        "length": 9.2
      }
    ]
  }
}
```

- `__typename` is a special meta-field that resolves to the name of the type.

---

## Input Object Types

- Input types are used to pass complex arguments into queries or mutations.

#### Schema

```graphql
input ReviewInput {
  stars: Int!
  commentary: String
}

type Mutation {
  createReview(episode: Episode, review: ReviewInput): Review
}
```

#### Query

```graphql
    mutation {
      createReview {
        episode: JEDI
        review: {
          stars: 5
          commentary: "This is a great movie!"
        }
      } {
        stars
        commentary
      }
    }
```

#### Response

```json
{
  "data": {
    "createReview": {
      "stars": 5,
      "commentary": "This is a great movie!"
    }
  }
}
```

---

## Directives

- Directives modify parts of a GraphQL schema or operation using a `@` character followed by the directive name.

### Type System Directives

- Used to annotate types, fields, and arguments for validation or execution changes.

```graphql
type User {
  fullName: String
  name: String @deprecated(reason: "Use 'fullName'.")
}

directive @deprecated(
  reason: String = "No longer supported!"
) on FIELD_DEFINITION | ENUM_VALUE
```

---

## Arguments

- Arguments can be optional or required and are passed by name.

```graphql
type <object_type> {
  length(unit: LengthUnit = METER): Float
}
```

- If the `unit` argument is not passed, it defaults to `METER`.

---

## Documentation

- Add descriptions to your schema for better clarity:
  - Multi-line: `""" <description> """`
  - Single-line: `" <description> "` or `# <description>`

---

<a href="../README.md">Prev: Home</a> | <a href="./queries.md">Next: Queries</a>
