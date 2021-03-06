---
title: Locations
caption: Type-safe Routing
category: servers
permalink: /servers/features/locations.html
feature:
  artifact: io.ktor:ktor-locations:$ktor_version
  class: io.ktor.locations.Locations
redirect_from:
- /features/locations.html
ktor_version_review: 1.0.0
---

{::options toc_levels="1..2" /}

Ktor provides a mechanism to create routes in a typed way, for both:
constructing URLs and reading the parameters.

Locations are an experimental feature.
{: .note.experimental}

**Table of contents:**

* TOC
{:toc}

{% include feature.html %}

## Installing the feature
{: #installing }

The Locations feature doesn't require any special configuration:

```kotlin
install(Locations)
```

## Defining route classes
{: #route-classes }

For each typed route you want to handle, you need to create a class (usually a data class)
containing the parameters that you want to handle.

The parameters must be of any type supported by the [Data Conversion](/servers/features/data-conversion.html) feature.
By default, you can use `Int`, `Long`, `Float`, `Double`, `Boolean`, `String`, enums and `Iterable` as parameters.

### URL parameters
{: #parameters-url }

That class must be annotated with `@Location` specifying
a path to match with placeholders between curly brackets `{` and `}`. For example: `{propertyName}`.
The names between the curly braces must match the properties of the class.

```kotlin
@Location("/list/{name}/page/{page}")
data class Listing(val name: String, val page: Int)
```

* Will match: `/list/movies/page/10`
* Will construct: `Listing(name = "movies", page = 10)`

### GET parameters
{: #parameters-get }

If you provide additional class properties that are not part of the path of the `@Location`,
those parameters will be obtained from the GET's query string or POST parameters:

```kotlin
@Location("/list/{name}")
data class Listing(val name: String, val page: Int, val count: Int)
```

* Will match: `/list/movies?page=10&count=20`
* Will construct: `Listing(name = "movies", page = 10, count = 20)`

## Defining route handlers
{: #route-handlers }

Once you have [defined the classes](#route-classes) annotated with `@Location`,
this feature artifact exposes new typed methods for defining route handlers:
`get`, `options`, `header`, `post`, `put`, `delete` and `patch`.

```kotlin
routing {
    get<Listing> { listing ->
        call.respondText("Listing ${listing.name}, page ${listing.page}")
    }
}
```

Some of these generic methods with one type parameter, defined in the `io.ktor.locations`, have the same name as other methods defined in the `io.ktor.routing` package. If you import the routing package before the locations one, the IDE might suggest you generalize those methods instead of importing the right package. You can manually add `import io.ktor.locations.*` if that happens to you.
Remember this API is experimental. This issue is already [reported at github](https://github.com/ktorio/ktor/issues/368).
{: .note}


## Building URLs
{: #building-urls }

You can construct URLs to your routes by calling `application.locations.href` with
an instance of a class annotated with `@Location`:

```kotlin
val path = application.locations.href(Listing(name = "movies", page = 10, count = 20))
```

So for this class, `path` would be `"/list/movies?page=10&count=20""`.

```kotlin
@Location("/list/{name}") data class Listing(val name: String, val page: Int, val count: Int)
```

If you construct the URLs like this, and you decide to change the format of the URL,
you will just have to update the `@Location` path, which is really convenient.

## Subroutes with parameters
{: #subroutes }

You have to create classes referencing to another class annotated with `@Location` like this, and register them normally:

```kotlin
routing {
    get<Type.Edit> { typeEdit -> // /type/{name}/edit
        // ...
    }
    get<Type.List> { typeList -> // /type/{name}/list/{page}
        // ...
    }
}
```
 
To obtain parameters defined in the superior locations, you just have to include
those property names in your classes for the internal routes. For example:

```kotlin
@Location("/type/{name}") data class Type(val name: String) {
    @Location("/edit") data class Edit(val type: Type)
    @Location("/list/{page}") data class List(val type: Type, val page: Int)
}
```
