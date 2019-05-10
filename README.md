# gatsby-source-mysql

[![version](https://img.shields.io/npm/v/gatsby-source-mysql.svg)](https://www.npmjs.com/package/gatsby-source-mysql) ![license](https://img.shields.io/npm/l/gatsby-source-mysql.svg)

Source plugin for pulling data into Gatsby from MySQL database.

## How to use

```javascript
// In your gatsby-config.js
module.exports = {
  plugins: [
    {
      resolve: `gatsby-source-mysql`,
      options: {
        connectionDetails: {
          host: 'localhost',
          user: 'db-username',
          password: 'db-password',
          database: 'world'
        },
        queries: [
          {
            statement: 'SELECT * FROM country',
            idFieldName: 'Code',
            name: 'country'
          }
        ]
      }
    }
    // ... other plugins
  ]
};
```

And then you can query via GraphQL with the type `allMysql<Name>` where `<Name>` is the `name` for your query.

Below is a sample query, however, it is probably different from yours as it would dependent on your configuration and your SQL query results.

Use [GraphiQL](https://www.gatsbyjs.org/docs/introducing-graphiql/) to explore the available fields.

```graphql
query {
  allMysqlCountry {
    edges {
      node {
        Code
        Name
        Population
      }
    }
  }
}
```

### multiple queries

When you have multiple queries, add another item in the `queries` option with different `name`.

```javascript
// In your gatsby-config.js
module.exports = {
  plugins: [
    {
      resolve: `gatsby-source-mysql`,
      options: {
        connectionDetails: {
          host: 'localhost',
          user: 'db-username',
          password: 'db-password',
          database: 'world'
        },
        queries: [
          {
            statement: 'SELECT * FROM country',
            idFieldName: 'Code',
            name: 'country'
          },
          {
            statement: 'SELECT * FROM city',
            idFieldName: 'ID',
            name: 'city'
          }
        ]
      }
    }
    // ... other plugins
  ]
};
```

### joining queries

It's possible to join the query for one-to-many relationship by providing `parentName` and `foreignKey` to the query object.

> one-to-one and many-to-many relationships are not supported currently. If you have a use case for them [raise an issue][raise-issue], and I'll look into it.

```javascript
// In your gatsby-config.js
module.exports = {
  plugins: [
    {
      resolve: `gatsby-source-mysql`,
      options: {
        connectionDetails: {
          host: 'localhost',
          user: 'db-username',
          password: 'db-password',
          database: 'world'
        },
        queries: [
          {
            statement: 'SELECT * FROM country',
            idFieldName: 'Code',
            name: 'country'
          },
          {
            statement: 'SELECT * FROM city',
            idFieldName: 'ID',
            name: 'city',
            parentName: 'country',
            foreignKey: 'CountryCode'
          }
        ]
      }
    }
    // ... other plugins
  ]
};
```

In the example above, `country` and `city` is one-to-many relationship (one country to multiple cities), and there are joined with `country.Code = city.CountryCode`.

With the configuration above, you can query a country joined with all the related cities with

```graphql
query {
  allMysqlCountry {
    edges {
      node {
        Code
        Name
        Population
        cities {
          Name
        }
      }
    }
  }
}
```

It also works the other way, i.e. you can query the country when getting the city

```graphql
query {
  allMysqlCity {
    edges {
      node {
        Name
        country {
          Name
        }
      }
    }
  }
}
```

## Plugin options

As this plugin is a wrapper of the popular [`mysql`](https://www.npmjs.com/package/mysql) library, the options are based on the library.

- **connectionDetails** (required): options when establishing the connection. Refer to [`mysql` connection options](https://www.npmjs.com/package/mysql#connection-options)
- **queries** (required): an array of object for your query. Each object could have the following fields:

| Field         | Required? | Description                                                                                                                               |
| ------------- | --------- | ----------------------------------------------------------------------------------------------------------------------------------------- |
| `statement`   | Required  | the SQL query statement to be executed.                                                                                                   |
| `idFieldName` | Required  | column that is unique for each record. This column must be returned by the `statement`.                                                   |
| `name`        | Required  | name for the query. Will impact the value for the graphql type                                                                            |
| `parentName`  | Optional  | name for the parent entity. In a one-to-many relationship, this field should be specified on the child entity (entity with many records). |
| `foreignKey`  | Optional  | foreign key to join the parent entity.                                                                                                    |

[raise-issue]: https://github.com/malcolm-kee/gatsby-source-mysql/issues/new
