## Schemas and Types

- Depending on libraries, types/fields/resolver functions might come built-in or allow you to define types/fields more ergonomically via schema definition language (SDL), then write the resolver functions for the corresponding fields separately.

  - Example:

  ```graphql
  type <object_type> {
      <field_1>: String! # non-null type
      <field_2>: [Episode!]!
  }
  ```

  - String is one of the built-in scalar types.
    - Scalar types cannot have sub-selections; they are "leaves" or "end" of the query.
      - Examples: Int, Float, String, Boolean, ID.
  - The exclamation mark guarantees non-null values or arrays with at least one element.

- Schemas must support query, mutation, and subscription operations.

  - Example:

  ```graphql
  schema {
    query: MyQueryType
    mutation: MyMutationType
    subscription: MySubscriptionType
  }
  ```

- Custom scalar types can be created in most GraphQL service implementations.
  - Example:
  ```graphql
  scalar Date
  ```
  - It is up to the implementation to define how the type should be serialized/deserialized and validated.
    - Example: Keeping a "Date" type as an integer timestamp.

### Enum Types

- Enum types define a set of expected outputs, reducing human error.
- Example:
  ```graphql
  enum Episode {
    NEWHOPE
    EMPIRE
    JEDI
  }
  ```

### Type Modifiers

- By default, types are nullable and singular in GraphQL.

#### Non-Null

- Example:

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

#### List

- Examples:

  ```graphql
  myField: [String!]  # Example 1

  myField: null       # Valid
  myField: []         # Valid
  myField: ['a', 'b'] # Valid
  myField: ['a', null, 'b'] # Invalid

  myField: [String!]  # Example 2

  myField: null       # Error
  myField: []         # Valid
  myField: ['a', 'b'] # Valid
  myField: ['a', null, 'b'] # Valid

  myField: [String!]! # Example 3

  myField: null       # Error
  myField: []         # Valid
  myField: ['a', 'b'] # Valid
  myField: ['a', null, 'b'] # Error
  ```

### Abstract Types: Interface Type

- Useful for returning objects or sets of objects that might be of several different types.

  - Example:

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

- Used to ask for fields on specific object types.
  - Example:
  ```graphql
  hero(episode: JEDI) {
    name
    ... on Droid {
      primaryFunction
    }
  }
  ```
  - The "hero" field returns a Character, which could be either "Human" or "Droid."

### Union Types

- Cannot define shared fields among the constituent types.

  - Example:

  ```graphql
  union SearchResult = Human | Droid | Starship

  # Operation
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

  # Response
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
  - Shared fields from a common interface can be queried once for multiple types.
    - Example:
    ```graphql
    search(text: "an") {
      __typename
      ... on Character {
        name
      }
      ... on Human {
        height
      }
      ... on Droid {
        primaryFunction
      }
      ... on Starship {
        name
        length
      }
    }
    ```

### Input Object Types

- Defined with the "input" keyword instead of "type."

  - Example:

  ```graphql
  # Schema
  input ReviewInput {
    stars: Int!
    commentary: String
  }

  type Mutation {
    createReview(episode: Episode, review: ReviewInput): Review
  }

  # Operation
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

  # Response
  {
    "data": {
      "createReview": {
        "stars": 5,
        "commentary": "This is a great movie!"
      }
    }
  }
  ```

### Directives

- Directives modify parts of a GraphQL schema or operation using a `@` character followed by the directive name.

  - "Type system directives" annotate types, fields, and arguments for validation or execution changes.

    - Example:

    ```graphql
    # For SDL-supported implementations
    type User {
      fullName: String
      name: String @deprecated(reason: "Use 'fullName'.")
    }

    # Underlying definition
    directive @deprecated(
      reason: String = "No longer supported!"
    ) on FIELD_DEFINITION | ENUM_VALUE
    ```

  - Specify where directives may be used, and custom directives can be defined.

### Arguments

- Arguments can be optional or required and are passed by name specifically.
  - Example:
  ```graphql
  type <object_type> {
    length(unit: LengthUnit = METER): Float
  }
  ```
  - If the "unit" argument is not passed, it defaults to METER.

### Documentation

- Add descriptions to your schema:
  - Multi-line: `""" <description> """`
  - Single-line: `" <description> "` or `# <description>`
