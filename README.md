# Where art thou, my error?

GraphQL is still in its early stages and thus these are very exciting times, indeed! Traditionally the GraphQL team has
taken the approach of defining the bare minimum in the specification that was deemed needed and otherwise letting the
community come-up with defining problems and experiment with solutions for those. One such example is how metadata about
the origins of errors that occurred during execution were added to the spec [TODO](link to issue).

This is great in the sense that we still have the ability, as a community, to shape the future of a GraphQL spec that we
all _want_ to use, but on the other hand it also means that we may need to spend significant amounts of time on thinking
about these problems and iterate. Seeing as we all strife to have backwards compatible schemas, it’s of great importance
that we know of the various iterations that people have experimented with and what the outcome was.

This is our story of working with errors.

## Exceptions vs User Errors

First of all, I wanted to take a step back and talk about errors in general. There are 2 types of errors that we have to
deal with:

1. Errors that we expect may occu

## Possible solutions

### Top level errors and treating a complete response as unusable when such errors exist

### Top level errors and working those back into the schema on the client-side

### Make errors part of schema as separate field on object that had error

### Make errors part of schema using a union

## Our choice and how we encode it into the schema

### Limitation of scalar types not fitting in unions

## Example of how we consume the errors in our schema
