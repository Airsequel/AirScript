# AirScript

A scripting language for spreadsheet formulas, CLI tools, ETL pipelines.

Aka can be used as transformers:
- cells → script → cell
- stdin | script | stdout
- read query → script → write query


## Ideas

- No side-effects (external data is stored in vars on start)
- Pure (and probably lazy)
- No modules/packages (but extensive stdlib)
- Hard resource constraints (runtime, memory, cycles)
- Typed (but no type annotations, only inference)
- No recursive functions (prevents endless loops)
- No currying
- JIT compiled (No overhead of managing both the source and the binary)
- List of available files (file content) in frontmatter
    - With support for globbing


## Observations

**Input data is untyped.**

- It's a lot of effort to generate all the necessary types
    to represent the data.
    (As seen in Elm and Haskell.)
- TypeScript shows how runtime type-checking can be very helpful here.


## Syntax

- Everything that starts with `$` (constants, variables, …)
    or starts with a capital letter (types, namespaces, …)
    is provided by the prelude (aka standard library)
- Syntax sugar for `f(g(h(i(j()))))`
    - Maybe `f@g@h@i@j()`
- `when … is …`
    - Syntax sugar for boolean:
      ```py
      when x is
      true "Good job!"
      false "Try again!"
      ```
- TODO: Maybe use `Yes`/`No` instead of `True`/`False`
    `YesNo = Yes | No`


### Types

- Number
    - `0`, `2`, …
    - `3.14`, …
    - TODO: Maybe `Rational` would be a better default
- Text
    - `"With spaces"`
    - `'single` (Also called atoms)
- Result (`Result ok = Ok(ok) | Error(Text)`)
    - No support for custom Error types,
      as one can use atoms and map them
      to full error messsages at the end
- Options (e.g. `Options['north, 'east, 'south, 'west]`)


## Examples

Manipulating JSON:

```py
---
description: Convert item prices from € to % of total price
tags: [json]
---

json_result = $parse_json($input)

items_result = when json_result is
  Null -> Error("Provide an Array and not Null")
  Object(hash_map) -> hash_map.items  # `.` on hash maps returns Result
  Array(items) -> Ok(items)
  _ -> Error("Provide an Array and not a primitive value")

items_total_price = Result:map(items_result,
    items -> items
      & List:map(item -> item.price & Result:withDefault(0))
      & List:sum
  )

# Final expression must be a `Result`
# - `Error`: E.g. printed to stderr, shown as cell error, …
# - `Ok`: E.g. printed to stdout (`Json` is automatically stringified),
#         shown as cell content, …
items_result
  & Result:map(items ->
      items & List:map(item ->
        item.price
          & Result:map(price -> price / items_total)
      )
    )
```


Instead of threading the errors through the whole program,
this can also be simplified to:

```py
---
description: Convert item prices from € to % of total price
tags: [json]
---

json_result = $parse_json($input)

items = when json_result is
  Array(items) -> items
  Null -> $stop(Error("Provide an Array and not Null"))
  Object(hash_map) -> $stop_error(hash_map.items)  # `.x` returns Result
  _ -> stop(Error("Provide an Array and not a primitive value"))

items_total_price = items
  & List:map(item -> item.price & Result:withDefault(0))
  & List:sum

Ok(items & List:map(item ->
  item.price & Result:map(price ->
    price / items_total_price
  )
))
```


## Related

### Alternative Programming Languages

Deal-breakers are marked with a ✋.

- [PRQL](https://prql-lang.org/)
    - Pipelined Relational Query Language
    - Compiles to SQL

- [Numbat](https://numbat.dev) (Rust)
    - Statically typed
    - For scientific computations
    - First class support for physical dimensions and units
    - ✋ No support for including external data

- [Roc](https://www.roc-lang.org) (Rust)
    - Statically typed
    - ✋ Side effects
