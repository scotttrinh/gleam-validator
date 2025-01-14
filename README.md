# Valid

![CI](https://github.com/sporto/gleam-validator/workflows/test/badge.svg?branch=main)

A validation library for [Gleam](https://gleam.run/).

API Docs: <https://hexdocs.pm/valid>.

This library follows the principle [Parse don't validate](https://lexi-lambda.github.io/blog/2019/11/05/parse-don-t-validate/).

## Install

```
gleam add valid
```

## Usage

You start with an input type and validate into an output type. These two types can be different. For example:

```haskell
type UserInput { UserInput(name: Option(String), age: Int) }

type ValidUser { ValidUser(name: String, age: Int) }
```

Then you create a validator like:

```haskell
import validator
import validator/common.{ValidatorResult}
import validator/int
import validator/option

fn user_validator(user: UserInput) -> ValidatorResult(ValidUser, String) {
  validator.build2(ValidUser)
  |> validator.validate(user.name, option.is_some("Please provide a name"))
  |> validator.validate(user.age, int.min(13, "Must be at least 13 years old"))
}
```

And run it:

```rust
case user_validator(input) {
  Ok(valid_user) -> ...
  Error(tuple(first_error, all_errors)) -> ...
}
```

## Error type

Errors can be your own type e.g.

```haskell
import validator
import validator/common.{ValidatorResult}
import validator/int
import validator/option

type Error {
  ErrorEmptyName,
  ErrorTooYoung,
}

fn user_validator(user: UserInput) -> ValidatorResult(ValidUser, String) {
  validator.build2(ValidUser)
  |> validator.validate(user.name, option.is_some(ErrorEmptyName))
  |> validator.validate(user.age, int.min(13, ErrorTooYoung))
}
```

## ValidatorResult

`ValidatorResult(valid, error)` is an alias for `Result(valid, tuple(error, List(error)))`

The `Ok` branch has the valid output.

The `Error` branch has a tuple `tuple(error, List(error))`.
The first value is the first error. The second value is a list with all errors (including the first).

## Validators

See the [API Docs](https://hexdocs.pm/gleam_validator/) for the list of included validators.

## Custom property validator

A property validator has two components:

- The error to return
- A function that transforms the property if successful (`fn(input) -> Option(output)`)

Example:

```rust
import gleam/option.{None, Option, Some}
import validator

fn bigger_than_10(num: Int) -> Option(num) {
  case num > 10 {
    True ->
      Some(num)
    False ->
      None
  }
}

let custom = validator.custom("Must be bigger than 10", bigger_than_10)

let validator = fn(form: FormInput) {
  validator.build1(ValidForm)
  |> validator.validate(form.quantity, custom)
}
```

## Examples

See [the tests](https://github.com/sporto/gleam-validator/blob/main/test/validator_test.gleam) for many examples

## Test

```sh
rebar3 compile
rebar3 eunit
```

## Installation

This package can be installed by adding `gleam_validator` to your `rebar.config` dependencies:

```erlang
{deps, [
    gleam_validator
]}.
```
