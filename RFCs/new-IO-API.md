# new IO API

## Moonbit's current IO API
Currently, the most commonly used IO APIs of Moonbit is:
```go
fn print[X: Show](x: X) {
  print_string(x.to_string())
}
fn println[X: Show](x: X) {
  print_string(x.to_string())
  print_string("\n")
}
```
where:
```go
interface Show {
  to_string(Self) -> String
}
```

## Problems with current API
The current API calls `to_string` unconditionally,
however, this would cause problems when printing `String` itself:

- we want `println((s : String))` to output `s` itself
- when we print a data types containing string fields,
for example in `derive(Show)`,
we want the strings to be printed with double quotes

To have correct `println` on `String`s, `to_string` of `String` must return the string itself.
To have correct derived `Show` implementation, we want `to_string` of `String` to add double quotes.
These two requirements are contradictory.

Worse, we cannot hack deriver of `Show` to treat `String` specially.
Because for types like `struct T[X] { x: X }`,
we want their derived `Show` to work correctly when `X` is instantiated to `String`.
So the current IO API cannot fulfill both requirements (`println` and `Show`) at the same time.

In essence, `String` serves two purpose in the language:
as the final representation of things to be printed (don't want double quotes),
and as a normal data type (need double quotes).
The current API does not distinguish between these two uses of `String`, which is the cause of the problem.

## The proposed new IO API

- rename the interface `Show` to `Debug` to make its purpose clearer.
Also rename the method `to_string` to `debug_to_string`
- `print` and `println` should only accept `String` as argument
- rename the old `println` to `debug` to avoid writing a lot of `println(bla.to_string())` boilerplates
