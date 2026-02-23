[GraphQL](https://graphql.org/) is a query language typically used by web `APIs` as an alternative to REST. It enables the client to fetch required data through a simple syntax while providing a wide variety of features typically provided by query languages, such as SQL. 

Like REST APIs, GraphQL APIs can read, update, create, or delete data. However, GraphQL APIs are typically implemented on a single endpoint that handles all queries. As such, one of the primary benefits of using GraphQL over traditional REST APIs is the efficiency in resource utilization and request handling.

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

Here is a `"general"` introspection query that dumps all information about types, fields, and queries supported by default : 
```graphql
query IntrospectionQuery {
      __schema {
        queryType { name }
        mutationType { name }
        subscriptionType { name }
        types {
          ...FullType
        }
        directives {
          name
          description
          
          locations
          args {
            ...InputValue
          }
        }
      }
    }

    fragment FullType on __Type {
      kind
      name
      description
      
      fields(includeDeprecated: true) {
        name
        description
        args {
          ...InputValue
        }
        type {
          ...TypeRef
        }
        isDeprecated
        deprecationReason
      }
      inputFields {
        ...InputValue
      }
      interfaces {
        ...TypeRef
      }
      enumValues(includeDeprecated: true) {
        name
        description
        isDeprecated
        deprecationReason
      }
      possibleTypes {
        ...TypeRef
      }
    }

    fragment InputValue on __InputValue {
      name
      description
      type { ...TypeRef }
      defaultValue
    }

    fragment TypeRef on __Type {
      kind
      name
      ofType {
        kind
        name
        ofType {
          kind
          name
          ofType {
            kind
            name
            ofType {
              kind
              name
              ofType {
                kind
                name
                ofType {
                  kind
                  name
                  ofType {
                    kind
                    name
                  }
                }
              }
            }
          }
        }
      }
    }
```

We can visualize the schema using the tool [GraphQL-Voyager](https://github.com/APIs-guru/graphql-voyager) that we have to self-host if we're working with confidential data, or we can use the [demo version](https://apis.guru/graphql-voyager/) otherwise.

We can then try to `query` some fields of the database to try to leak sensitive information.

# IDOR 
Using `GraphQL` queries, we should try to access data we are not authorized to access, or data that can be accessed without authorization. It could be user information enumeration for example.

# SQL Injection
`SQL` injection can inherently occur in GraphQL APIs that do not properly sanitize user input from arguments in the SQL queries. 

- To identify if a query requires an argument, we can send the query without any arguments and analyze the response.

As the GraphQL query only returns the first row, we need to use the `GROUP_CONCAT` function to exfiltrate multiple rows at a time. This enables us to exfiltrate all table names in the current database with the following payload:
```sql
{
  user(username: "x' UNION SELECT 1,2,GROUP_CONCAT(table_name),4,5,6 FROM information_schema.tables WHERE table_schema=database()-- -") {
    username
  }
}
```

# XSS 
`XSS vulnerabilities` can occur if GraphQL responses are inserted into the HTML page without proper sanitization.

`XSS vulnerabilities` can also occur if invalid arguments are reflected in error messages, if an `XSS payload` is reflected without proper encoding in the GraphQL error message.

# DoS
We can try to create some `GraphQL queries` that result in exponentially large responses, requiring significant resources to process. This can lead to high hardware utilization, and thus DoS.

- We can try to identify a `loop` between fields. 

