# Mutation

## Overview

- GraphQL specification states that "the resolution of fields other than top-level ("data"), mutation fields must always be side effect-free and idempotent"

## Adding New Data

#### Schema

```graphql
enum Episode {
  NEWHOPE
  EMPIRE
  JEDI
}

input ReviewInput {
  stars: Int!
  commentary: String
}

type Mutation {
  createReview(episode: Episode, review: ReviewInput!): Review
}
```

#### Operation

```graphql
mutation createReviewForEpisode($ep: Episode!, $review: ReviewInput!) {
  createReview(episode: $ep, review: $review) {
    stars
    commentary
  }
}
```

#### Variable

```json
{
  "ep": "JEDI",
  "review": {
    "stars": 5,
    "commentary": "This is a great movie!"
  }
}
```

- A hypothetical function that writes a new review to a database during a "createReview" mutation might look something like:

```javascript
    Mutation: {
        createReview(obj, args, context, info){
            return context.db
                .createNewReview(args.episode, args.review)
                .then( (reviewData) => new Review(reviewData) )
        }
    }
```

---

## Updating Data

#### Schema

```graphql
type Mutation {
  updateHumanName(id: ID!, name: String!): Human
}
```

#### Operation

```graphql
mutation UpdateHumanName($id: ID!, $name: String!) {
  updateHumanName(id: $id, name: $name) {
    id
    name
  }
}
```

#### Variable

```json
{
  "id": "100",
  "name": "Luke Starkiller"
}
```

#### Response

```json
{
  "data": {
    "id": "1000",
    "name": "Luke Starkiller"
  }
}
```

- As a general rule, a GraphQL API should be designed to help clients get and modify data in a way that makes sense for them, so the fields defined in a schema should be informed by these use cases

---

## Removing Data

#### Schema

```graphql
type Mutation {
  deleteStarship(id: ID!): ID!
}
```

#### Operation

```graphql
    mutation DeleteStarship(id: ID!){
        deleteStarship(id: $id)
    }
```

#### Variable

```json
{
  "id": "3003"
}
```

#### Response

```json
{
  "data": {
    "deleteStarship": "3003"
  }
}
```

- While query fields are executed in parallel, **mutation fields run in series**. It's different from the notion of a "database transaction"
  - For example, there's no built-in support to revert a successful operations while other operations fail. Thus, a GraphQL API response may return "data" object and "errors" object for corresponding requests
