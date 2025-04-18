## How validation works

1. GraphQL operation reaches the server
2. Document is first parsed
3. Then, the document is validated to the system

---

## Execution: How GraphQL provides data for requested fields

## Field Resolvers

#### schema

```graphql
type Query {
  human(id: ID!): Human
}
```

- resolver function example:
  ```javascript
  function Query_human(obj, args, context, info) {
    return context.db
      .loadHumanById(args.id)
      .then((userData) => new Human(userData));
  }
  ```
- `obj`: previous object
- `context`: value provided by every resolver may hold contextual info about logged-in user, db, etc.

```graphql
type Human {
  name: String
  appearsIn: [Episode]
  starships: [Starship]
}

enum Episode {
  NEWHOPE
  EMPIRE
  JEDI
}

type Starship {
  name: String
}
```

#### operation

```graphql
query {
  human(id: 1002) {
    name
    appearsIn
    starships {
      name
    }
  }
}
```

- Each field on each type is backed by a resolver function that's written by the GraphQL server developer. When a field is executed, the corresponding resolver is called to produce the next value.

- During execution, GraphQL will wait for Promises to be completed before continuing and will do so with optimal concurrency.

---

## Scaler Coercison

- It's a term used for conversion of values into the types expected by the schema
- The type system knows what to expect and will convert the values returned by a resolver function into something that upholds the API contract
- For example:

```graphql
    Human: {
        appearsIn(obj){
            return obj.appearsIn # return [4,5,6]
        }
    }
```

- In this example, there may be an Enum type defined on the server that uses numbers like 4, 5 and 6 internally, but represents them as the expected values in the GraphQL type system

---

## List Resolvers

- It returns a list of Promises
- For example:

```javascript
function Human_starships(obj, args, context, info) {
  return obj.starshipIDs.map((id) =>
    context.db.loadStarshipByID(id).then((shipData) => new Starship(shipData))
  );
}
```

- GraphQL will wait for all of these promises concurrently before continuing, and when left with a list of objects, it will continue yet again to load "name" field on each of these items concurrently.

---

<a href="./subscriptions.md">Prev: Subscriptions</a> | <a href="./response.md">Next: Response</a>
