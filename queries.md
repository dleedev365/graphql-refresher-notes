## Queries: Fetching data from a GraphQL Server

- A root operation type (Query, Mutation, Subscription) is the entry point to the API. Then, selection set of fields are defined and execution continues until leaf values are returned. The final result is returned on a top-level, "data"

  - ex.

  ```graphql
      {
          "data": {
              <resposne>
          }
      }
  ```

  - GraphQL can traverse related objects and their fields, so lots of releated data are fetched in one request
  - In GraphQL, every field and nested object can get its own set of arguments (including Scalar types)

- Query example,

```graphql
    query <operation_name>{
        hero {
            name
            friends{
                name
            }
        }
    }


```

    - "aliases" can be used
        - ex.
        ```graphql
            query {
                empireHero: hero(episode: EMPIRE){
                    name
                }
                jediHero: hero(episode: JEDI){
                    name
                }
            }
        ```

## Variables

- A first-class way to factor dynamic values out of the query and pass them as a separate dictionary
- Rules:
  1. Replace static value in the query with `$<variableName>`
  2. Declare `$variableName` as one of the variables accepted by the query
  3. Pass `variableName: <value>` in the separate, transport-specific (like JSON) variables dictionary
- ex.

```graphql
    // oepration
    query hero(episode: $episode){
        name
        friends{
            name
        }
    }

    // variables
    {
        "episode": "JEDI"
    }
```

    - all declared variables must be Scalar, Enum or Input Object types

## Fragments

- Reusable units that let you contruct sets of fields and include them in queries when needed
- The concept is used to split complicated application data requirements into smaller chunks, especially, when you need to combine many UI components with different fragments into one initial data fetch

  - ex.

  ```graphql
      query {
          leftComparision: hero(episode: EMPIRE){
              ...comparisionFields
          }
          rightComparison: hero(episode: JEDI){
              ...comparisionFields
          }
      }

      fragment comparisionFields on Character{
          name
          appearsIn
          friends: {
              name
          }
      }
  ```

  - It's also possible for fragements to access variables declared in the operation

    - ex.

    ```graphql
        query HeroComparision($first: Int = 3){
            leftComparision: hero(episode: EMPIRE){
            ...comparisionFields
        }
            rightComparison: hero(episode: JEDI){
                ...comparisionFields
            }
        }

        fragement comparisionFields on Character{
            name
            firendsConnection(first: $first){
                <field>
            }
        }
    ```

## Inline Fragments

- It's useful when you need to return unioon type or interface. It can access data on the underying concrete type

  - ex.

  ```graphql
      // operation
      query HeroForEpisode($ep: Episode!){
          hero(episode: $ep){
              name
              ... on Droid{
                  primaryFunction
              }
              ... on Human{
                  height
              }
          }
      }

      // variables
      {
          "ep": "JEDI"
      }

      // response
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

- To dynamically chage the structure and shape of queries using variables (when executed **on the server**)
- "executable directives" can be attached to a field or fragment inclusion by a client, and can affect execution of the query in any way the server desires.

  - It can be useful for situations where you'd need to do string manipulation to add/rename fields in your query.

    - ex. `@include(if: Boolean)`, `@skip(if: Boolean)`
    - ex.

    ```graphql
        // operation
        query Hero($episode: Episode, $withFriends: Boolean!){
            hero(episode: $Episode){
                name
                friends @include(if: $withFriends){
                    name
                }
            }
        }

        // variables
        {
            "episode": "JEDI",
            "withFriends": false
        }

        // response
        {
            "data": {
                "hero": {
                    "name" : "R2-D2"
                }
            }
        }
    ```
