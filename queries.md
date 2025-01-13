# Queries: Fetching Data from a GraphQL Server

- A root operation type (Query, Mutation, Subscription) is the entry point to the API. Then, a selection set of fields is defined, and execution continues until leaf values are returned. The final result is returned on a top-level `data`.

  ```json
  {
      "data": {
          "<response>"
      }
  }
  ```

  - GraphQL can traverse related objects and their fields, allowing related data to be fetched in one request.
  - In GraphQL, every field and nested object can have its own set of arguments (including scalar types).

- **Query Example**

  ```graphql
  query <operation_name> {
      hero {
          name
          friends {
              name
          }
      }
  }
  ```

- **Using Aliases**

  ```graphql
  query {
    empireHero: hero(episode: EMPIRE) {
      name
    }
    jediHero: hero(episode: JEDI) {
      name
    }
  }
  ```

## Variables

- A first-class way to factor dynamic values out of the query and pass them as a separate dictionary.

- **Rules**:

  1. Replace the static value in the query with `$<variableName>`.
  2. Declare `$variableName` as one of the variables accepted by the query.
  3. Pass `variableName: <value>` in the separate, transport-specific (e.g., JSON) variables dictionary.

  ```graphql
  # Operation
  query hero($episode: Episode!) {
      hero(episode: $episode) {
          name
          friends {
              name
          }
      }
  }

  # Variables
  {
      "episode": "JEDI"
  }
  ```

- All declared variables must be scalar, enum, or input object types.

## Fragments

- Reusable units that let you construct sets of fields and include them in queries when needed.
- Useful for splitting complex application data requirements into smaller chunks, especially when combining multiple UI components.

  ```graphql
  query {
    leftComparison: hero(episode: EMPIRE) {
      ...comparisonFields
    }
    rightComparison: hero(episode: JEDI) {
      ...comparisonFields
    }
  }

  fragment comparisonFields on Character {
    name
    appearsIn
    friends {
      name
    }
  }
  ```

- Fragments can also access variables declared in the operation:

  ```graphql
  query HeroComparison($first: Int = 3) {
      leftComparison: hero(episode: EMPIRE) {
          ...comparisonFields
      }
      rightComparison: hero(episode: JEDI) {
          ...comparisonFields
      }
  }

  fragment comparisonFields on Character {
      name
      friendsConnection(first: $first) {
          <field>
      }
  }
  ```

## Inline Fragments

- Useful when you need to return a union type or interface. Inline fragments can access data on the underlying concrete type.

  ```graphql
  # Operation
  query HeroForEpisode($ep: Episode!) {
      hero(episode: $ep) {
          name
          ... on Droid {
              primaryFunction
          }
          ... on Human {
              height
          }
      }
  }

  # Variables
  {
      "ep": "JEDI"
  }

  # Response
  {
      "data": {
          "hero": {
              "name": "R2-D2",
              "primaryFunction": "Astromech"
          }
      }
  }
  ```

## Directives

- Directives dynamically change the structure and shape of queries using variables when executed **on the server**.
- "Executable directives" can be attached to a field or fragment inclusion and affect the query execution as desired by the server.

  - Examples: `@include(if: Boolean)`, `@skip(if: Boolean)`.

  ```graphql
  # Operation
  query Hero($episode: Episode, $withFriends: Boolean!) {
      hero(episode: $episode) {
          name
          friends @include(if: $withFriends) {
              name
          }
      }
  }

  # Variables
  {
      "episode": "JEDI",
      "withFriends": false
  }

  # Response
  {
      "data": {
          "hero": {
              "name": "R2-D2"
          }
      }
  }
  ```
