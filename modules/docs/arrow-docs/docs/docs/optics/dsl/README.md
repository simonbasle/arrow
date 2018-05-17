---
layout: docs
title: Syntax DSL
permalink: /docs/optics/syntax/
---

## Optics DSL

Arrow offers a Optics DSL to compose different Optics while improving ease of use and readability.
To avoid boilerplate Arrow will generate this property-like dsl using `@optics` annotation.

```kotlin
package com.example.domain

@optics data class Street(val number: Int, val name: String)
@optics data class Address(val city: String, val street: Street)
@optics data class Company(val name: String, val address: Address)
@optics data class Employee(val name: String, val company: Company?)
```

The DSL will be generated in the same package as your `data class` and can be used on the `Companion` of your class.

```kotlin:ank
import com.example.domain.*
import arrow.optics.dsl.*

val john = Employee("John Doe", Company("Kategory", Address("Functional city", Street(42, "lambda street"))))

val optional: Optional<Employee, String> = Employee.company.address.street.name
optional.modify(john, String::toUpperCase)
```

Arrow can also generate dsl for a `sealed class` which can be helpful to reduce boilerplate code, or improve readability.

```kotlin
package com.example.domain

@optics sealed class NetworkResult
@optics data class Success(val content: String): NetworkResult()
@optics sealed class NetworkError : NetworkResult()
@optics data class HttpError(val message: String): NetworkResult()
object TimeoutError: NetworkError()
```

Let's imagine we have a function `f` of type `(HttpError) -> HttpError` and we want to invoke it on the `NetworkResult`.

```kotlin:ank
val networkResult: NetworkResult = HttpError("boom!")
val f: (String) -> String = String::toUpperCase

when (networkResult) {
  is HttpError -> networkResult.copy(f(networkResult.message))
  else -> networkResult
}
```

We can rewrite this code with our generated dsl.

```kotlin:ank
NetworkResult.networkError.httpError.message.modify(networkResult, f)
```

The DSL also has special support for [Each]({{ '/docs/optics/each' | relative_url }}) and [At]({{ '/docs/optics/at' | relative_url }}).

`Each` can be used to focus into a structure `S` and see all its foci `A`. Here we focus into all `Employee`s in the `Employees`.

```kotlin
@optics data class Employees(val employees: ListK<Employee>)
```

```kotlin:ank
import arrow.data.*

val jane = Employee("Jane Doe", Company("Kategory", Address("Functional city", Street(42, "lambda street"))))
val employees = Employees(listOf(john, jane).k())

Employees.employees.every(ListK.each()).company.address.street.name.modify(employees, String::capitalize)
```

If you are in the scope of `Each` you don't need to specify the instance.

```kotlin:ank
ListK.each<Employee>().run {
  Employees.employees.every.company.address.street.name.modify(employees, String::capitalize)
}
```

`At` can be used to focus in `A` at a given index `I` for a given structure `S`.

```kotlin
@optics data class Db(val content: MapK<Int, String>)
```

Here we focus into the value of a given key in `MapK`.

```kotlin:ank
val db = Db(mapOf(
  1 to "one",
  2 to "two",
  3 to "three"
).k())

Db.content.at(MapK.at(), 2).some.modify(db, String::reversed)
```

If you are in the scope of `At` you don't need to specify the instance.

```kotlin:ank
MapK.at<Int, String>().run {
  Db.content.at(2).some.modify(db, String::reversed)
}
```