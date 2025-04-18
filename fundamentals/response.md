## Response: How GraphQL Returns Data to Clients

- Syntax example:

```json
    {
        "data": {...},
        "errors": [
            {...}
        ]
    }
```

- GraphQL implementations may include additional arbitrary info about the response in the "extension" key
  - It's reserved for GraphQL implementation to provide additional info about the response such as telemetry data, rate limit consumption, etc.
