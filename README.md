# Where art thou, my error?

GraphQL is still in its early stages and thus these are very exciting times, indeed! Traditionally the GraphQL team has
taken the approach of defining the bare minimum in the specification that was deemed needed and otherwise letting the
community come-up with defining problems and experiment with solutions for those. One such example is how metadata about
the origins of errors that occurred during execution were [added to the specification][spec-errors-locations].

This is great in the sense that we still have the ability, as a community, to shape the future of a GraphQL spec that we
all _want_ to use, but on the other hand it also means that we may need to spend significant amounts of time on thinking
about these problems and iterate. Seeing as we all strife to have backwards compatible schemas, it’s of great importance
that we know of the various iterations that people have experimented with and what the outcome was.

This is our story of thinking about and working with errors, thus far.

NOTE: Throughout this talk I’ll use ‘query execution’ to indicate executing a GraphQL document, be it a query or
      mutation operation. I have a hard time relating to ‘document execution’, mostly because I don’t see others using
      it, but perhaps I’ve just missed it. Come at me, at the bar, and set me straight!

## Errors vs errors

First of all, I wanted to take a step back and talk about errors in general. The nomenclature around these can get
confusing, suffice to say that during this session we’ll talk about these two types:

1. Errors that occurred during query execution that were unexpected and _could_ lead to corrupted data. We’ll refer to
   these as (top-level) ‘GraphQL errors’, going forward.

   These could be due to hardware failures, such as running out of memory or disk space, network failures, or unexpected
   upstream data etc.

   When these occur, `graphql-js` will return `null` for the field that triggered the error and serialize the error into
   the top-level `errors` list, next to the successful response `data`. (Presumably other implementations
   follow this reference implementation.)

   ```json
   {
     data: { artwork: { artist: { name: "Vincent van Gogh", leftEarSize: null } } },
     errors: [
       {
         message: "An unexpected error occurred",
         path: ["artwork", "artist", "leftEarSize"]
       }
     ]
   }
   ```

1. Exceptions to these are errors that are known to occur and are expected to be handled by the user of an API. We’ll
   refer to these as ‘exceptions’, going forward.

   By default these are treated equally by `graphql-js` to top-level GraphQL errors, if uncaught.

We will **not** be speaking about errors that occur _outside_ of query execution, such as network failures reaching the
GraphQL server, parsing a syntactically incorrect document, or passing variables that don’t satisfy the type-system; as
these will all lead to a query being rejected wholesale and are solve-able using traditional means, such as a `4xx` HTTP
status code.

## What is the problem we’re trying to solve?

Because with GraphQL we’re usually requesting data for multiple resources, there may be a situation where some fields
resolve successfully and some may fail. This is also why, when using an HTTP transport layer, the advice is to always
respond with a HTTP 200 (ok) status. Determining how to process the response is left up to the client.

So how _do_ we model errors in such a way that they can be meaningful and in context of their origin?

* What if you want to render partial data?

  - Maybe the failed data is unrelated to other components that you were also requesting data for.

    [TODO: Insert screenshot of page that has a Recently Viewed artwork rail]

  - Or the data that failed was part of a list and other entries can still be rendered just fine.

    [TODO: Insert screenshot of artwork grid]

* Or what if you’d (additionally) like to communicate the error in your interface?

  - Perhaps you’d want to display that some functionality only works when authorized.

    [TODO: Come up with example on artsy and insert screenshot]

  - Or the query is in response to a mutation and you’d like to communicate input validation failures.

    [TODO: Insert screenshot of order pages]

## Possible solutions

### Top-level GraphQL errors and treating a complete response as unusable when such errors exist

Some clients, such as Apollo and Relay Classic, have made the decision to reject a response entirely by default if any
top-level GraphQL errors exist. This is because clients can really only fully assume that the response data is
incomplete, not wether or not your application could handle that case.

This may be an ok solution when you’re starting out or all the requested data is part of one holistic view, but it
quickly breaks down when you want a little more than that.

### Top-level GraphQL errors with extra metadata

GraphQL errors only have a single field in [the specification][spec-errors] to provide context around the cause of the
error, which is the `message` field. However, [the specification][spec-response] also defines an `extensions` key, which
may hold a map of freeform data for the schema implementors to extend the protocol however they see fit.

Apollo Server 2.0, for instance, [introduced standardized errors][apollo-server-errors] you can throw from your
resolvers, which end up being serialized into the `extensions` map. An example they give is for bad user input:

```js
import { UserInputError } from 'apollo-server';

const resolvers = {
  Query: {
    events(root, { zipCode }) {

      // do custom validation for user inputs
      const validationErrors = {};
      if (!isValidZipCode(zipCode)) {
        validationErrors.zipCode = 'This is not a valid zipcode';
      }
      if (Object.keys(validationErrors).length > 0) {
        throw new UserInputError(
          'Failed to get events due to validation errors',
          { validationErrors }
        );
      }
      // actually query events here and return successfully
      return getEventsByZipcode(zipCode);
    }
  }
}
```

Seeing as these extensions are freeform, however, this builds an **implicit** contract between the server and client
that then needs to be abstracted away by additional client code, which is unfortunate, when you think about it, because
GraphQL is meant to explicitly express shapes of data.

The Apollo team acknowledges this by adding:

> While convenient, the weakness of this approach is that the format of the validation error messages is not captured by
> your schema, making it brittle to changes. Unless you maintain tight control of both server and client, you should
> keep the error responses as simple as possible.
>
> For mutations, it can be worthwhile defining these validation errors as first class citizens within your schema.

(Which we’ll address next.)

### Make (mutation) error metadata part of schema as separate fields

One [commonly suggested approach][apollo-mutation-responses] around mutations is to define status metadata on the
response type next to the field of the affected entity. For example, a response type could look like:

```graphql
type UpdateArtworkMutationResponse {
  success: Boolean!
  message: String!
  artwork: Artwork
}
```

Here there’s a boolean that indicates success, an extra message that sheds context on the situation when a failure
occurs, and finally the `user` that an update was attempted to be made to.

Adding these fields to the same namespace makes sense when we’re thinking of the failure case, but what about the
success case? Do we really need a `success` boolean to indicate that updates to the `artwork` were made? What purpose
serves the `message` field, other than possibly being a sign of a very friendly schema that sends you happy messages?

Finally, this approach only really works for mutations, as they get their own root type to start a query from. It would
be hard to imagine how to apply this to queries.

### Make error metadata part of schema as separate field

Similarly, [another suggested approach][error-fields] is to add an additional `error` field to the type in question,
which then describes the error that occurred. The previous example could be rewritten like so:

```graphql
type GenericError {
  message: String!
}

type UpdateArtworkMutationResponse {
  error: GenericError
  artwork: Artwork
}
```

If `error` is not `null`, something went wrong. This cleans up the namespace a bit, but more importantly this approach
can be applied to queries too:

```graphql
type PublishedArtworkNotification {
  artwork: Artwork!
}

type PublishedArtworkNotificationsPayload {
  error: GenericError
  notifications: [PublishedArtworkNotification]
}

type Query {
  publishedArtworkNotificationsPayload: PublishedArtworkNotificationsPayload!
}
```

Neat.

If you don’t control the server schema and are using a client that can extend a schema, you could try to retrofit these
error fields into the schema on the client side based on the top-level GraphQL errors, as shown [here][retrofit-errors].

## Recap

So to quickly recap, ideally we want a solution to:

* utilize GraphQL to explicitly describe the error data
* present the error data exactly where the error occurred in the schema
* work for both mutations and queries
* be concise and encourage ‘clean’ typed; that is, no pollution of namespaces with fields only needed in some cases

### Make exceptions first-class citizens of your schema

To that end, the final approach we’ll be discussing, and the one that we at Artsy have started adopting, is to give
exceptions their own type and return those instead of the success type, when they occur. To do this we make use of a
union of both the success and the exception type (or multiples thereof) and then query for those.

The benefits are:

* You can further model the exception in an explicit and introspect-able way.

  For example, in the case of an upstream HTTP failure, your exception type could include an integer status-code field
  and document it as such.

  ```graphql
  type Artwork {
    title: String!
  }

  type HTTPError {
    message: String!
    statusCode: Int!
  }

  union ArtworkOrError = Artwork | HTTPError

  type Query {
    artworkOrError(id: ID!): ArtworkOrError
  }
  ```

  ```graphql
  query {
    artworkOrError("mona-lisa") {
      ... on Artwork {
        title
      }
      ... on HTTPError {
        statusCode
      }
    }
  }
  ```

* You know exactly where the exception occurred in the graph.

  ```graphql
  type Artist {
    artworksOrErrors: [ArtworkOrError!]!
  }

  type Query {
    artist(id: ID!): Artist
  }
  ```

  ```graphql
  query {
    artist("leonardo-da-vinci") {
      artworksOrErrors {
        ... on Artwork {
          title
        }
        ... on HTTPError {
          statusCode
        }
      }
    }
  }
  ```

* You can use it for both mutations and queries.

  ```graphql
  type UpdateArtworkMutationResponse {
    artworkOrError: ArtworkOrError
  }
  ```

* All fields will always be captured in the single `artworkOrError` field and if no information about the error is
  needed you simply don’t query for it and get back `null` instead.

  ```graphql
  query {
    artworkOrError("mona-lisa") {
      ... on Artwork {
        title
      }
    }
  }
  ```

## How we encode it into our schema

I should preface this by clearly stating that while have been thinking about this problem for a while now, only recently
have we started rolling these changes out into our schema, so some of these are not yet discoverable in [our open-source
GraphQL service][metaphysics].

### Types

As shown before, we define a union of the actual result type _and_ the error type. However, we additionally (will)
define a set of error interfaces, which make it possible for clients to query for errors in a more generic way.

```graphql
interface Error {
  message: String!
}

interface HTTPError {
  message: String!
  statusCode: Int!
}

type HTTPErrorType implements Error, HTTPError {
  message: String!
  statusCode: Int!
}

type Artwork {
  title: String!
}

union ArtworkOrError = Artwork | HTTPErrorType

type Query {
  artworkOrError(id: ID!): ArtworkOrError
}
```

We can now still query as shown in the earlier examples:

```graphql
query {
  artworkOrError("mona-lisa") {
    ... on Artwork {
      title
    }
    ... on HTTPError {
      message
      statusCode
    }
  }
}
```

…but we can now also have generic error components that would query like so:

```graphql
query {
  artworkOrError("mona-lisa") {
    ... on Artwork {
      title
    }
    ... on Error {
      ...GenericErrorComponent
    }
  }
}

fragment GenericErrorComponent on Error {
  message
}
```

For the record, we have _not_ yet put these interfaces into production, so the nomenclature is not set in stone yet and
I’d love to hear your input on this. Is `Error` _too_ generic to use as the base error type? Is there a nicer pattern
to avoid having to suffix concrete types of an error interface with `Type`?

Side-note, there’s [an RFC][implements-interface-rfc] to the GraphQL specification that would make it possible to have
interface implement other interfaces, thus removing the need to keep repeating the fields of super-sets. This RFC has
recently been moved to being a draft, yay!

### Naming

As you may have noticed, we’re calling these fields `something` _or_ `error`. We are mostly doing this to stay backwards
compatible with out existing schema. While we could certainly add exception types to existing union fields, we can’t
change a single type field into a union type field without breaking compatibility.

Instead we now may have 2 versions of a given field:

* one with the single type field which is nullable, in case an exception occurred

  ```graphql
  query {
    artwork("mona-lisa") {
      title
    }
  }
  ```

* and another that has the error union type

  ```graphql
  query {
    artworkOrError("mona-lisa") {
      ... on Artwork {
        title
      }
      ... on HTTPError {
        statusCode
      }
    }
  }
  ```

This duplication is slightly unfortunate, from a clean schema design perspective, but it’s not a very uncommon pattern
in the community. For instance, many schemas provide 2 ways to retrieve lists:

* one as an immediate list:

  ```graphql
  query {
    artworks: [artwork]
  }
  ```

* and one as a ‘connection’ (as defined by the [Relay Connection specification][relay-connection-spec])

  ```graphql
  type ArtworkEdge {
    node: Artwork
  }

  type ArtworkConnection {
    edges: [ArtworkEdge]
  }

  query {
    allArtworks: ArtworkConnection
  }
  ```

So the jury is still out on wether or not that’s a bad way to name things. We’ll have to see after using this for a
while.

### Downside of using a union

One notable downside is that GraphQL scalar types can _not_ be included in unions, [right now][scalars-in-unions]. Thus,
if you have scalar fields that could lead to exceptions, you will have to ‘box’ those in object types.

```graphql
type ArtworkPurchasableBox {
  value: Boolean!
}

union ArtworkPurchasableOrError = ArtworkPurchasableBox | HTTPError

type Artwork {
  currentlyPurchasableOrError: ArtworkPurchasableOrError!
}
```

This is definitely a case where the pattern of defining 2 fields, one with and one without exception types, comes in
handy. Having to always query through the box type is inelegant, to put it softly.

### Show example of factory code that produces both single and union typed fields

TODO

### Example of how we consume the errors in our clients

TODO

[spec-errors]: https://facebook.github.io/graphql/draft/#sec-Errors
[spec-response]: https://facebook.github.io/graphql/draft/#sec-Response-Format
[spec-errors-locations]: https://github.com/facebook/graphql/pull/230
[apollo-mutations-responses]: https://www.apollographql.com/docs/guides/schema-design.html#mutation-responses
[error-fields]: https://itnext.io/the-definitive-guide-to-handling-graphql-errors-e0c58b52b5e1
[retrofit-errors]: https://github.com/facebook/relay/issues/1913#issuecomment-358636018
[apollo-server-errors]: https://blog.apollographql.com/full-stack-error-handling-with-graphql-apollo-5c12da407210
[metaphysics]: http://github.com/artsy/metaphysics
[relay-connection-spec]: https://facebook.github.io/relay/graphql/connections.htm
[scalars-in-unions]: https://github.com/facebook/graphql/issues/215
[implements-interface-rfc]: https://github.com/facebook/graphql/pull/373
