## Best Practices

- GraphQL schema describes "how" clients use the data, rather than mirroring the legacy database schema

---

## APIs

- All entities in GraphQL are not identified by URLs. Instead, a GraphQL server operates on a single URL/endpoint, "/graphql", and all GraphQL requests for a given service should be directed to this endpoint.
- The server should handle authentication before a GraphQL request is validated.
- Authorization should happen within your business logic during GraphQL request execution
- Client should indicate a media type on the "Accept" of "application/graphql-response+json"

- There're other best practices on GraphQL API design you can find on the official documentation

---

## Authorization

- It's suggested that authrization logic to be delegated to the business logic layer
- For example:

#### schema/document

```graphql
type Post {
  authorId: ID!
  body: String
}
```

#### Authorization logic

```javascript
    export const postRepository = {
        getBody ( {user, post} ){
            if (user?.id && (user.id === post.authorId)){
                return post.body
            }
        }
        return null
    }
```

#### Business logic

```javascript
import { postRepository } from "postRepository";

function Post_body(obj, args, context, info) {
  return postRepository.getBody({ user: context.user, post: obj });
}
```

- With "type system directives", authorization can be in the GraphQL layer as well (not recommended)

  - For example:

  ```graphql
  directive @auth(rule: Rule) on FIELD_DEFINITION

  enum Rule {
    IS_AUTHOR
  }

  type Post {
    authorId: ID!
    body: String @auth(rule: IS_AUTHOR)
  }
  ```

---

## Pagination

- Pagination can be implemented in two ways: `offset`-based and `cursor`-based

---

## Offset Pagination

- Example:

```graphql
    friends(first: 2, offset: 2) # grabs the next two in the list
    friends(first: 2, after: $firendId) # grabs the next two after the last friend fetched
    friends(first: 2, offset: $firendCursor) #  where we get a cursofr from the last item, use that to paginate
```

---

## Cursor Pagination

- It can be done through "edges" (new concept)
- For example:

#### interfaces

```graphql
interface Character {
  id: ID!
  name: String!
  friends: [Character]
  friendsConnection(first: Int, after: ID): FriendsConnection!
  appearsIn: [Episode]!
}

type FriendsConnection {
  totalCount: Int
  edges: [FriendsEdge]
  friends: [Character]
  pageInfo: PageInfo!
}

type FriendsEdge {
  cursor: ID!
  node: Character
}

type PageInfo {
  starCursor: ID
  endCursor: ID
  hasNextPage: Boolean!
}
```

#### query

```graphql
query {
  hero {
    name
    friends(first: 2) {
      edges {
        node {
          name
        }
        cursor
      }
    }
  }
}
```

- To keep track of "end-of-list", "counts", and "connections", the earlier query can be further developed as follows:

```graphql
query {
  hero {
    friends(first: 2) {
      totalCount
      edges {
        node {
          name
        }
        cursor
      }
      pageInfo {
        endCursor
        hasNextPage
      }
    }
  }
}
```
