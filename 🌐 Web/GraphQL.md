[GraphQL](https://graphql.org/) is a query language typically used by web `APIs` as an alternative to REST. It enables the client to fetch required data through a simple syntax while providing a wide variety of features typically provided by query languages, such as SQL. Like REST APIs, GraphQL APIs can read, update, create, or delete data. However, GraphQL APIs are typically implemented on a single endpoint that handles all queries. As such, one of the primary benefits of using GraphQL over traditional REST APIs is the efficiency in resource utilization and request handling.

We can identify the GraphQL engine used by the web application using the tool [graphw00f](https://github.com/dolevf/graphw00f). It will send various GraphQL queries, including malformed ones, to try to identify the GraphQL engine used.
```bash
# -f to fingerprint, -d to use detect mode
python3 main.py -d -f -t http://172.17.0.2
```

# Introspection 
[Introspection](https://graphql.org/learn/introspection/) is a GraphQL feature that enables users to query the GraphQL API about the `structure of the backend system`. As such, users can use introspection queries to obtain all queries supported by the API schema. These introspection queries query the `__schema` field.

We can identify all GraphQL types supported by the backend using the following query:
```graphql
{
  __schema {
    types {
      name
    }
  }
}
```

Furthermore, we can obtain all the queries supported by the backend using this query:
```graphql
{
  __schema {
    queryType {
      fields {
        name
        description
      }
    }
  }
}
```

Knowing all supported queries helps us identify potential attack vectors that we can use to obtain sensitive information. Lastly, we can use the following "general" introspection query that dumps all information about types, fields, and queries supported by the backend:

We can visualize the schema using the tool [GraphQL-Voyager](https://github.com/APIs-guru/graphql-voyager) that we have to self-host if we're working with confidential data, or we can use the [demo version](https://apis.guru/graphql-voyager/) otherwise.

