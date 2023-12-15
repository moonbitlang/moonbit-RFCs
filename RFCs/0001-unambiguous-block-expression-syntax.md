# Summary
Today's block expression syntax ambiguously overlaps with struct literal shorthand field syntax. This RFC proposes requiring the `do` keyword (or another) before block expressions to disambiguate them.

# The Problem
Given this type:

```rust
struct Knight {
  name: String
}
```

## OK
```rust
fn Knight::new(given_name: String) -> Knight {
  let name = {
    let prefix = "Sir"
    "\(prefix) \(given_name)"
  }

  { name: name }
  // Expr Type Mismatch
  //     has type : String
  //     wanted   : Person
}
```

## ERROR (treats it as a block, rather than a shorthand struct literal)
```rust
fn Knight::new(given_name: String) -> Knight {
  let name = {
    let prefix = "Sir"
    "\(prefix) \(given_name)"
  }

  { name }
  // Expr Type Mismatch
  //     has type : String
  //     wanted   : Person
}
```

## OK
```rust
fn Knight::new(given_name: String) -> Knight {
  let name = {
    let prefix = "Sir"
    "\(prefix) \(given_name)"
  }

  { name, }
}
```

# Solution
The problem goes away if the parser was able to diambiguate between blocks and struct literals. The most obvious way would be to require a keyword (or sigil) before blocks, since they are less commonly used than struct literals. The `do` keyword is a natural choice, and one that has either been discussed or chosen by existing languages like JavaScript, Perl, etc.

```rust
fn Knight::new(name: String) -> Knight {
  // this is clearly a block
  let name = do {
    let prefix = "Sir"
    "\(prefix) \(given_name)"
  }

  // this is clearly a shorthand struct literal
  { name }
}
```

# Problems
The most obvious problem is whichever keyword (or sigil) is chosen likely can only be used for this purpose. The keyword `do` in particular is sometimes used for other meanings in languages, like `do` notation [in Haskell](https://en.wikibooks.org/wiki/Haskell/do_notation) which while vaguely similar, is not the same thing.

# Prior Art

- [Perl](https://perldoc.perl.org/functions/do): `do { ... }`
- [JavaScript](https://github.com/tc39/proposal-do-expressions) TC39 Proposal: `do { ... }`
- [C++](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2023/p2806r1.html#prior-art) WG21 Proposal: `do { ... }`
- [C GCC](https://gcc.gnu.org/onlinedocs/gcc/Statement-Exprs.html) Non-standard extension: `({ ... })`
