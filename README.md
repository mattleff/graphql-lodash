# GraphQL Lodash

![GraphQL Lodash logo](docs/gqlodash-logo.png)

Unleash power of [lodash](https://lodash.com/) inside your GraphQL queries

## Why?
GraphQL allows to ask for what you need and get exactly that. But what about the shape?
GraphQL Lodash gives you the power of `lodash` right inside your GraphQL Query using `@_` directive.

[![lodash usage gif](docs/lodash.gif)](https://apis.guru/graphql-lodash/)

**Note**: This is an **experimental** project created to explore the concept of **Query and transformation collocation**.

We encourage you to try it inside our [demo](https://apis.guru/graphql-lodash/). Here are a few query examples you can run against StartWars API:

#### Find out a plannet with the biggest population
```
{
  planetWithMaxPopulation: allPlanets @_(get: "planets") {
    planets @_(maxBy: "population") {
      name
      population
    }
  }
}
```
#### Get gender statistics
```
{
  genderStats: allPeople @_(get: "people") {
    people @_(countBy: "gender") {
      gender
    }
  }
}
```
#### Map characters to films they are featured in
```
{
  peopleToFilms: allPeople @_(get: "people") {
    people @_(
      keyBy: "name"
      mapValues: "filmConnection.films"
    ) {
      name
      filmConnection {
        films @_(map: "title") {
          title
        }
      }
    }
  }
}
```

## Install

    npm install --save graphql-lodash
or

    yarn add graphql-lodash

## API

### `graphqlLodash(query, [operationName])`

- **query** (_required_) - query string or query AST
- **operationName** (_optional_) - required only if the query contains multiple operations

### Returns
```
{
  query: string|object,
  transform: Function
}
```
- **query** - the original query with stripped `@_` directives
- **transform** - function that recieves `response.data` as a single argument and returns
the same data in the intended shape.



## Usage Examples

The simplest way to integrate `graphql-lodash` is to write wrapper function for graphql client of you choice:
```js
import { graphqlLodash } from 'graphql-lodash';

function lodashQuery(queryWithLodash) {
  let { query, transform } = graphqlLodash(queryWithLodash);
  // Make a GraphQL call using 'query' variable as a query
  // And place result in 'result' variable
  ...
  result.data = transform(result.data);
  return result;
}
```

### Fetch example
An example of a simple client based on [fetch API](https://developer.mozilla.org/en/docs/Web/API/Fetch_API):
```js
function executeGraphQLQuery(url, query) => {
  return fetch(url, {
    method: 'POST',
    headers: new Headers({"content-type": 'application/json'}),
    body: JSON.stringify({ query: query })
  }).then(responce => {
    if (responce.ok)
      return responce.json();
    return responce.text().then(body => {
      throw Error(responce.status + ' ' + responce.statusText + '\n' + body);
    });
  }
}

function lodashQuery(queryWithLodash) {
  let { query, transform } = graphqlLodash(queryWithLodash);
  return executeGraphQLQuery(query).then(result => {
    result.data = transform(result.data);
    return result;
  });
}
```

### Caching clients
For caching clients like Relay and Apollo we recomend to apply the transformation after the caching layer.
Here is proposed solution for Relay:

![Relay usage](docs/relay-architecture.png)

We are still figuring out how to do this and any [feedback](https://github.com/APIs-guru/graphql-lodash/issues/new) is welcome.

## Usage on server side

In theory this tool can be used on the server. But this will break the contract and, most likely,
will break all the GraphQL tooling you use. Use it on server-side only if you know what you do.