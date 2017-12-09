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

_show a very simple example of query analyzer and mention sangria/graphql-ruby_

In [GraphQL Ruby][graphql-ruby], [query analyzers][graphql-ruby-analysis] reuse concepts of [`Array#reduce`][array-reduce].
They visit each node in the query AST and can accumulate data during the visits. At the end of the visit, they can perform
an action with the final value.

_show a very simple example of a query analyzer that counts how many times an `id` field is used_

[Sangria][sangria] has similar functionality called [`QueryReducer`][sangria-query-reducer] and
[`SchemaBasedDocumentAnalyzer`][sangria-document-analyzer].

Here are a few interesting use cases of ahead-of-time query analysis.

__Tracking field usage__

One feature of GraphQL that is sometimes overlooked is the fact that the server knows exactly what parts of the schema
is actively used and by who.

This information is invaluable when you want to make a breaking change to your schema, such as removing a deprecated field.

A query analyzer can be used to walk the query AST and record the name of each type and each field that is being used
by the query. This information can then be saved in an external system which can be made queryable by schema maintainers.

The advantage of capturing this information at the analysis step versus the execution step is that we will record the
usage of fields even if they don't end up getting resolved.

[picture of query]

For example, in the above query if the `User` doesn't have any `repositories`, the `node` field will never get resolved, nor will the
`Repository.name` or `Repository.forkCount`, since `edges` will be an empty array.

This doesn't stop our query analyzer from recording the use of `Repository.name` and `Repository.forkCount` since the analysis
happens before execution, not during.

__Ahead-of-time permission checking__

__Calculating query complexity__

[part-1]: https://medium.com/@cjoudrey/life-of-a-graphql-query-lexing-parsing-ca7c5045fad8
[part-2]: https://medium.com/@cjoudrey/life-of-a-graphql-query-validation-18a8fb52f189
[graphql-ruby]: http://graphql-ruby.org
[graphql-ruby-analysis]: http://graphql-ruby.org/queries/analysis.html
[array-reduce]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/reduce
[sangria]: http://sangria-graphql.org
[sangria-query-reducer]: http://sangria-graphql.org/learn/#verification-with-query-reducers
[sangria-document-analyzer]: http://sangria-graphql.org/learn/#schema-based-document-analyzer
