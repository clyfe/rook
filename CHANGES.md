## 0.1.16 - UNRELEASED

* Namespace paths can now be a simple string as an alternative to a vector. E.g. `(rook/namespace-handler ["users" 'org.example.resources.users])`.

[Closed issues](https://github.com/AvisoNovate/rook/issues?q=is%3Aissue+milestone%3A0.1.16+is%3Aclosed)

## 0.1.15 - 3 Oct 2014

* The `new` and `edit` convention names for resource handler functions were removed.
* The convention name for PUT :id has changed from `update` to `change` (to avoid future conflict with Clojure 1.7).
* The :route-spec metadata key has been renamed to just :route.
* A new and very alpha integration with [ring-swagger](https://github.com/metosin/ring-swagger) has been introduced.

[Closed issues](https://github.com/AvisoNovate/rook/issues?q=is%3Aissue+milestone%3A0.1.15+is%3Aclosed)

## 0.1.14 - 15 Aug 2014

A critical bug related to dispatch and route parameters has been identified and fixed.

[Closed issues](https://github.com/AvisoNovate/rook/issues?q=milestone%3A0.1.14+is%3Aclosed)

## 0.1.13 - 28 Jul 2014

This release addresses a few minor issues related to reporting errors. 
Importantly, when using response validation, any 5xx error responses 
(usually indicating a failure inside the response handler function, or downstream from the function) are passed through unchanged.

[Closed issues](https://github.com/AvisoNovate/rook/issues?q=milestone%3A0.1.13+is%3Aclosed)

## 0.1.12 - 21 Jul 2014

This release updates a few dependencies, and adds additional debugging inside the io.aviso.rook.dispatcher namespace:

* With debug enabled, there is a message identifying how each incoming request is matched to a function
* With trace enabled, there is a message (at startup) identifying the merged metadata for each resource handler function

The merged metadata is the merge of the function's metadata with the containing namespace's. An attempt is made to eliminate common keys (such as :doc, :line, etc.) so that it's just the custom metadata provided on the function itself (or inherited from the namespace).

This extra debugging is very handy for diagnosing issues such as "is it invoking my handler?" or "why is my middleware not getting invoked?".

No issues were closed in this release.

## 0.1.11 - 8 Jul 2014

This release significantly revamped argument resolvers, including making the list of argument resolvers extensible using options to the namespace-handler, and via :arg-resolvers metadata on functions and namespaces.

Middleware for namespaces is no longer simple Ring middleware; the middleware is passed both the handler to wrap and the merged metadata for the resource handler function. This encourages the middleware to only wrap handlers for which it applies, leading to improved runtime efficiency. A new function, compose-middleware makes it easy to string together several middleware expressions, similar to how -> is used for Ring middleware.

In addition, it is now possible to add metadata defining response status code and corresponding body schemas; this is useful in development to ensure that your resource handler functions are returning the values you expect.

No issues were closed for this release.

## 0.1.10 - 1 Jul 2014

The major change in this release is the introduction of a new dispatcher system that scales larger and operates more efficiently than Compojure; in fact Compojure and Clout are no longer dependencies of Rook.

[Documentation for Rook](http://howardlewisship.com/io.aviso/documentation/rook) has been greatly expanded and moved out of the project.

We've also gone a long way towards improved efficiency; there's a new and improved system for matching resource handler function arguments to a _resolver_ that provides the value for the argument. This is now done computed once, when building the dispatcher, rather than computed by a search every time a resource handler function is invoked.

Expect some more changes in 0.1.11 that close the final loops in dynamic argument resolution, as well as making the argument resolution more extensible.

[Closed Issues](https://github.com/AvisoNovate/rook/issues?q=milestone%3A0.1.11+is%3Aclosed)

## 0.1.9 - 15 May 2014

This release refines the async support in Ring considerably; it replaces the odd 'return `false`' behavior for asynchronous handlers with the more traditional 'return `nil`' (to close the result channel).

Synchronous handlers are now explicitly invoked in a new `thread` block; previously they may have been executed inside a `go` block thread.

There's new middleware for supporting Ring sessions in a fully async pipeline.

*All* request handler arguments are resolved uniformly via the `:arg-resolvers` list; this includes previously hard-coded arguments such as `request`.

The default list of argument resolvers now includes the ability to resolve Ring request headers, for example: An argument named `content-type` will map to the `"content-type"` Ring request header.

Request handler function arguments may now be destructured maps; as long as the `:as` keyword is there (to provide a name for argument resolution). This is useful for extracting a large number of values from the Ring request `:params` map.

A default argument resolver for `params*` will resolve to the same as `params`, but with the map keys _Clojurized_ (underscores replaced with dashes).

A default argument resolver for `resource-uri` has been added; this is typically used to supply a `Location` header in a response.

Schema cooercion now understands converting strings to booleans, instants, and UUIDs. Schema validation reporting is better, but still a work in progress. Validation has been simplified; there's no longer any attempt to 'shave' the parameters to match the schema, so you will often need to add a mapping of `s/Any` to `s/Any` to prevent spurious failures (from arbitrary query parameters, for example).

__Note: there have been a number of refactorings; a few functions have been renamed, and in some places, key/value varargs have been replaced with a simple map.__

* namespace-middleware --> wrap-namespace
* arg-resolver-middleware --> wrap-with-arg-resolvers

[Closed Issues](https://github.com/AvisoNovate/rook/issues?q=milestone%3A0.1.9+is%3Aclosed)

## 0.1.8 - 31 Mar 2014

This release introduces a significant new feature: Asynchronous request processing using [core.async](https://github.com/clojure/core.async). At its core is the definition of an _asynchronous handler_ (or middleware), which accepts a Ring request map and returns its response inside a channel; asynchronous handlers are typically implemented using `go` blocks.

Resources within the same server will often need to cooperate; Rook supports this via the _asynchronous loopback_, which is a way for one resource to send a request to another resource using core.async conventions (and without involving HTTP or HTTPs).

There's also support for leveraging Jetty continuations so that all request handling is fully non-blocking, end-to-end.

Other features:

*  Validation of incoming request parameters using [Prismatic Schema](https://github.com/prismatic/schema)
* `io.aviso.rook.client` namespace to streamline cooperation between resources via the asynchronous loopback
* Metadata from the containing namespace is merged into metadata for individual resource handler functions

[Closed Issues](https://github.com/AvisoNovate/rook/issues?q=milestone%3A0.1.8+is%3Aclosed)
