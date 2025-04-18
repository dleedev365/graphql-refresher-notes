# Subscriptions

## Overview

- Subscription is about getting real-time updates from a GraphQL Server via long-lived requests
- GraphQL subscriptions are typically backed by a separate pub/sub system so that messages about updated data can be published as needed during runtime and then consumed by the resolver functions for subscription fields in the API (usually via WebSockets or SSE)

#### Subscrition Schema

```graphql
type Subscription {
  reviewCreated: Review
}

subscription NewReviewCreated {
  reviewCreated {
    rating
    commentary
  }
}
```

- A document may contain a number of different subscription operations with different root fields, but each operation must have exactly one root field

#### Operation

```graphql
    subscription {
        reviewCreated{
            rating
            summary
        }
    }

    humanFriendsUpdated{
        name
        friends{
            name
        }
    }
```

#### Document

```graphql
subscription NewReviewCreated {
  reviewCreated {
    reviewCreated {
      rating
      summary
    }
  }
}

subscription FriendListUpdated($id: ID!) {
  humanFriendsUpdated(id: $id) {
    name
    friends {
      name
    }
  }
}
```

---

<a href="./mutations.md">Prev: Mutations</a> | <a href="./validations_and_execution.md">Next: Validations & Execution</a>
