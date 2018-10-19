# Where art thou,
# my error?
### _@alloy - artsy.net_
#### https://github.com/alloy/graphql-finland

[.background-color: #FFFFFF]

^
- I’m Eloy
- Engineer at Artsy
- We use GraphQL since 2015

----

# Still early days

^
- Exciting
- Spec is minimal, let
  community define problems

----

# Still early days

![right, fit](images/spec-errors-locations.png)

^
- Error path is example

----

# Still early days

^
- We can still shape future
- Much thinking and
  experimenting
- Because backwards compat
  we should share iterations
- This is our story on errors

----

# 🙉

^
- Will use query execution
- document execution doesn’t
  sound natural
- Come at me, at bar

----

# Errors _vs_ Errors

^
- Errors in general
- It gets confusing

[.background-color: #FFFFFF]

----

# Errors _vs_ Errors

^
- Won’t talk about errors
  outside query execution
- Such as:
* network failure reaching
  GraphQL server
* syntax error
* variables that don’t satisfy
  type-system

[.background-color: #FFFFFF]

----

# Errors _vs_ Errors

^
- These all reject query entirely
- Solve using traditional means
- e.g. HTTP 4xx/5xx

[.background-color: #FFFFFF]

----

# Errors _vs_ Errors

^
- We’ll talk about 2

[.background-color: #FFFFFF]

----

# GraphQL Errors

^
- Unexpected
- Could lead to corrupted data

----

# GraphQL Errors

^
- Causes:
* Incorrect API usage
* hardware failures (mem, disk)
* upstream network failures
* unexpected upstream data

----

# GraphQL Errors

![right, fit](code/grapqh-response-with-errors.png)

^
- field resolves to null
- top-level error added

----

# Exceptions

^
- Known to occur
- Expected to be handled
  by user of API
- Treated same as GraphQL Errors
  if uncaught

----

# What’s the problem?

[.background-color: #FFFFFF]

^
- GraphQL fetches data for
  multiple resources
- Some fields resolve,
  some fail
- Advice for transport is
  always HTTP 200
- How to process errors
  is left to client

----

# What’s the problem?

[.background-color: #FFFFFF]

^
- So how to model errors
- meaningfully
- in context of origin

----

# Partial data

^
- What if you want
  to render?

----

# Partial data

![right, fit](images/partial-data-unrelated.png)

^
- Unrelated components

----

# Partial data

![right, fit](images/partial-data-unrelated-annotated.png)

^
- Unrelated components

----

# Partial data

![right, fit](images/partial-data-list.png)

^
- Part of list
- Others can render fine

----

# Partial data

![right, fit](images/partial-data-list-annotated.png)

^
- Part of list
- Others can render fine

----

# Communicate error

^
- What if you want
  to show in UI?

----

### Communicate error

![right, fit](images/mutation-validation-error.png)

^
- Mutation validation errors

----

# Possible solutions

[.background-color: #FFFFFF]

----

# GraphQL Errors
# and reject entire
# response

_https://u.nu/atrh_
_https://u.nu/pjv-_

^
- Apollo, Relay Classic
- Clients can only assume
  data is incomplete
- Not if app can handle
  partial data

----

# GraphQL Errors
# with metadata

_https://u.nu/r5-2_

^
- Errors only have message
  context field
- Spec also has extensions
- Free for schema implementor

----

# GraphQL Errors
# with metadata

_https://u.nu/r5-2_

^
- Apollo Server 2 uses it
- Throw error from resolver
- Serialized into extensions

----

### GraphQL Errors
### with metadata

![right, fit](code/graphql-errors-with-metadata.png)

^
- eg bad input

----

# GraphQL Errors
# with metadata

_https://u.nu/r5-2_

^
- But, freeform
- Implicit contract server & client
- Needs abstracted client code
- Unfortunate, GraphQL should be
  used to express shape of data

----

# GraphQL Errors
# with metadata

# [fit] **While convenient, the weakness of this approach is that the format of the validation error messages<br />is not captured by your schema, making it brittle to changes. Unless you maintain tight control<br />of both server and client, you should keep the error responses as simple as possible.<br />For mutations, it can be worthwhile defining these validation errors as first class citizens within your schema.**

^
- Apollo team acknowledges
- Which we’ll describe next

----

# Extra (mutation)
# error fields

_https://u.nu/th7y_

^
- Approach for mutations
- Metadata on response type

----

#### Extra (mutation)
#### error fields

![right, fit](code/extra-mutation-error-fields.png)

^
- bool for status
- message for context
- the artwork

----

#### Extra (mutation)
#### error fields

![right, fit](code/extra-mutation-error-fields.png)

^
- adding here makes sense
  for failures
- but what about success?
* do we need a bool?
* why have message?
* (besides feel-good message)

----

# Extra (mutation)
# error fields

_https://u.nu/th7y_

^
- only works well with mutations
- their return type is a query root
- so can pollute namespace

----

# Extra error
# field & type

_https://u.nu/-3pw_

^
- Similar, but also for queries
- Add error field to type

----

### Extra error
### field & type

![right, fit](code/extra-error-field-and-type-1.png)

^
- when error != null, failure
- cleans-up namespace

----

### Extra error
### field & type

![right, fit](code/extra-error-field-and-type-2.png)

^
- also useable on queries
- neat
- but in our case we don’
  have partial data
- we want either success or fail
- this approach would lead to
  either field being null
- or worse, ambiguous

----

# Extra error
# field & type

_https://u.nu/-3pw_
_Side-note: https://u.nu/i0yg_

^
- when not in control of server
- client can extend schema
- retrofit GraphQL Errors
- based on path field

----

# Recap

[.background-color: #FFFFFF]

----

# Recap

[.background-color: #FFFFFF]

* Use GraphQL

^
utilize GraphQL to explicitly describe the error data

----

# Recap

[.background-color: #FFFFFF]

* Use GraphQL
* In context

^
present the error data where the error occurred in the schema

----

# Recap

[.background-color: #FFFFFF]

* Use GraphQL
* In context
* All operations

^
work for both mutations and queries

----

# Recap

[.background-color: #FFFFFF]

* Use GraphQL
* In context
* All operations
* Explicit status

^
be concise and encourage ‘clean’ types; that is, no pollution of namespaces with fields only needed in some cases

----

# Exceptions as
# first-class citizens

^
- ticks all the boxes
- we started adopting this
- exceptions are own type
- union with success type
- query explicitly
- benefits are:

----

#### Exceptions as
#### first-class citizens

![right, fit](code/exceptions-as-first-class-citizens-1.png)

^
- model explicit
- introspect-able
- HTTP error to upstream service
  can have status-code field
- document as such

----

#### Exceptions as
#### first-class citizens

![right, fit](code/exceptions-as-first-class-citizens-2.png)

^
- in context

----

#### Exceptions as
#### first-class citizens

![right, fit](code/exceptions-as-first-class-citizens-3.png)

^
- works for all operations

----

#### Exceptions as
#### first-class citizens

![right, fit](code/exceptions-as-first-class-citizens-4.png)


^
- single field
- either success _or_ failure
- no pollution if not interested in error

----

# How we use it

[.background-color: #FFFFFF]

^
- preface that we just started
- not all exist in prod schema

----

# Types

![right, fit](code/how-we-use-it-types-1.png)

^
- as shown before
- union of success and error

----

# Types

![right, fit](code/how-we-use-it-types-2.png)

^
- additional interfaces
- clients can treat errors generically

----

# Types

![right, fit](code/how-we-use-it-types-3.png)

^
still query as shown before

----

# Types

![right, fit](code/how-we-use-it-types-4.png)

^
clients can now have generic components

----

# Types

![right, fit](code/how-we-use-it-types-5.png)

^
- no interfaces in prod yet
- is `Error` too generic?
- other naming pattern that avoids
  `...Type`?

----

# Types

_Side-note: https://u.nu/c1ye_

^
- RFC interfaces implement interfaces
- removed need to repeat
- has traction
- yay!

----

# Field naming

‘something’ _or_ error

^
- fields are named like this
- for us backwards compat
- we can extend existing unions
- but can’t change to unions
- instead we have 2 fields

----

# Field naming

![right, fit](code/how-we-use-it-field-naming-1.png)

^
one with single nullable type

----

# Field naming

![right, fit](code/how-we-use-it-field-naming-2.png)

^
another with error union type

----

# Field naming

![right, fit](code/how-we-use-it-field-naming-3.png)

^
- slightly unfortunate re clean design
- not uncommon, eg connection
- we’ll see how this feels

----

### Downside of union

![right, fit](code/how-we-use-it-downside-of-union.png)

^
- notable downside
- union can’t contain scalar
- needs to be boxed

----

### Downside of union

![right, fit](code/how-we-use-it-downside-of-union.png)

^
- useful that we have 2 fields
- one with and one without error
- querying through box is inelegant

----

# Downside of union

_Side-note: https://u.nu/c29p_

^
- RFC re scalar in union
- stage 0, needs champion
- we may, after using this

----

#### Example of query usage

![right, fit](code/example-of-usage-1.png)

----

#### Example of mutation usage

![right, fit](code/example-of-usage-2.png)

----

# Final thoughts

[.background-color: #FFFFFF]

🏁

^
- again, only just started
- but much thought/experimentation
- it addresses our needs

----

# Final thoughts

[.background-color: #FFFFFF]

📣 @alloy

^
- want feedback
- especially when adopting

----

# Final thoughts

[.background-color: #FFFFFF]

👫📈🤩

^
- as community openly iterate together
- as we try to make future of GraphQL great

----

# Final thoughts

[.background-color: #FFFFFF]

…to ‘REST’ 😉

^
- and put legit questions to rest

----

### **@alloy**
### **artsy.net**
#### https://github.com/alloy/graphql-finland

![right, fit](images/tweet-from-leeb.png)

^
- leave you with message from internet rando

