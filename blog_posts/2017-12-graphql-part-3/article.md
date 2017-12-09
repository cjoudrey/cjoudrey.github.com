---
tags: [GraphQL, Javascript, Ruby, Rails, Query, Validation, Analysis, AST]
---

# Life of a GraphQL Query - Ahead-of-time query analysis

This is the third part of an ongoing series I'm writing about the lifecycle of a GraphQL query.

If you've missed the previous parts, check them out here:

- [Life of a GraphQL Query - Lexing/Parsing][part-1]
- [Life of a GraphQL Query - Validation][part-2]

We previously left off with the AST of a validated query. In other words, we know the query is both syntactically correct
and that it can be executed against our schema.

Before diving into query execution, I wanted to write a short post about ahead-of-time query analysis.

With the AST of the validated query, we know what types and fields are being requested before executing the query.

[picture of AST]

Some GraphQL libraries provide a step between the validation and execution of a query called analysis.

[picture of steps]

A query analyzer can provide logic for validating incoming queries and reject them if they don't pass some criteria.
Analyzing a query doesn't _have_ to be about validating a query though. You could simply want to to record useful
information about the query before executing it.

Here are a few interesting use cases of ahead-of-time query analysis.

*Tracking field usage*

One feature of GraphQL that is sometimes overlooked is the fact that the server knows exactly what parts of the schema
is actively used and by who.

This information is invaluable when you want make a breaking change to your schema such as removing a deprecated field.

A query analyzer can be used to walk the query AST

*Ahead-of-time permission checking*

*Calculating query complexity*

[part-1]: https://medium.com/@cjoudrey/life-of-a-graphql-query-lexing-parsing-ca7c5045fad8
[part-2]: https://medium.com/@cjoudrey/life-of-a-graphql-query-validation-18a8fb52f189
[graphql-ruby]: http://graphql-ruby.org/queries/analysis.html
[sangria]: http://sangria-graphql.org/learn/#verification-with-query-reducers
