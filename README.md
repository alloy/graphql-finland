# Where art thou, my error?

GraphQL is still in its early stages and thus these are very exciting times, indeed! Traditionally the GraphQL team has
taken the approach of defining the bare minimum in the specification that was deemed needed and otherwise letting the
community come-up with defining problems and experiment with solutions for those. One such example is how metadata about
the origins of errors that occurred during execution were added to the spec [TODO](link to issue).

This is great in the sense that we still have the ability, as a community, to shape the future of a GraphQL spec that we
all _want_ to use, but on the other hand it also means that we may need to spend significant amounts of time on thinking
about these problems and iterate. Seeing as we all strife to have backwards compatible schemas, it’s of great importance
that we know of the various iterations that people have experimented with and what the outcome was.

This is our story of thinking about and working with errors, thus far.

## Errors vs errors

First of all, I wanted to take a step back and talk about errors in general. The nomenclature around these can get
confusing, suffice to say that during this session we’ll talk about these two types:

1. Errors that occurred during query execution that were unexpected and _could_ lead to corrupted data.

   These could be due to hardware failures, such as running out of memory or disk space, network failures, or unexpected
   upstream data etc.

   When these occur, `graphql-js` will return `null` for the field that triggered the error and serialize the error into
   the top-level ‘GraphQL errors’ list. Presumably other implementations follow this reference implementation.
   (TODO: check specification)

1. Exceptions to these are errors that are known to occur and are expected to be handled by the user. We’ll refer to
   these as ‘exceptions’, going forward.

   By default these are treated equally by `graphql-js` to other errors, if uncaught.

## What is the problem we’re trying to solve?

## Possible solutions

### Top level errors and treating a complete response as unusable when such errors exist

### Top level errors and working those back into the schema on the client-side

### Make errors part of schema as separate field on object that had error

### Make errors part of schema using a union

## Our choice and how we encode it into the schema

### Limitation of scalar types not fitting in unions

## Example of how we consume the errors in our schema
