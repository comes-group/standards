# ComesTalk Core

| | |
| --- | --- |
| **Name** | ComesTalk Core Library Specification |
| **Version** | 2021.1 pre-release |
| **Authors** | liquidev |

The Polish part of le Polish Smalltalk.

This document describes the core library of ComesTalk: the basic set of methods that are required to be implemented by all implementations of the language.

# Built-in types

The following paragraphs describe built-in types, that is, types tightly coupled with the language implementation. These types cannot be implemented in the language itself.

## Fundamental messages

These messages are implemented by all values of all types.

### `istnieje`

- **English name:** _exists_
- **Arguments:** none
- **Returns:** boolean

Returns whether this value is `nic` (nothing).

### `typ`

- **English name:** _type_
- **Arguments:** none
- **Returns:** class

Returns the class of this value.

### `wypisz`

- **English name:** _print_
- **Arguments:** none
- **Returns:** nothing

Prints the value to the console.
- Nothing shall be printed as `nic`.
- Booleans shall be printed as `tak` and `nie` for `true` and `false` respectively.
- Numbers shall be printed in base-10. The decimal precision is implementation-dependent, but should be optimized towards human readability rather than precision.
- Objects where this message is not overridden shall be printed as `<obiekt>`.
- Other built-in types shall provide implementation-defined ways of printing themselves, with the added constraint that each type must be printed in a distinct way, between angle brackets `<>`.

### `lub:`

- **English name:** _or_
- **Arguments:** `lub:x`, where x is any value
- **Returns:** (described below)

If this value is nothing, returns `x`. Otherwise returns this value.

## Nothing type

- **Type name:** `Nic`
- **Literals:** `nic`

### `nie`

- **English name:** _not_
- **Arguments:** none
- **Returns:** boolean

Returns _true_ (`tak`).

## Boolean type

- **Type name:** `Logiczna`
- **Literals:** `tak` (_true_) and `nie` (_false_)

### `nie`

- **English name:** _not_
- **Arguments:** none
- **Returns:** boolean

Boolean NOT; returns _false_ (`nie`) if this value is _true_ (`tak`), or _true_ (`tak`) if this value is _false_ (`nie`).

### `i:`

- **English name:** _and:_
- **Arguments:** `i:x`, where x is a boolean
- **Returns:** boolean

Boolean AND; returns _true_ (`tak`) if, and only if, both this value and `x` are _true_ (`tak`). Otherwise returns _false_ (`nie`).

### `lub:`

- **English name:** _or:_
- **Arguments:** `lub:x`, where x is a boolean
- **Returns:** boolean

Boolean OR; returns _true_ (`tak`) if this value is _true_, or `x` is _true_. If both values are _false_, returns _false_.

### `tak:`

- **English name:** _true:_
- **Arguments:** `tak:wtedy`, where `wtedy` is a function
- **Returns:** Whatever `wtedy` returns after executing it.

Conditionally executes `wtedy` without any arguments, if this value is _true_.

### `nie:`

- **English name:** _false:_
- **Arguments:** `nie:wtedy`, where `wtedy` is a function
- **Returns:** Whatever `wtedy` returns after executing it.

Conditionally executes `wtedy` without any arguments, if this value is _false_.

### `tak:nie:`

- **English name:** _true:false:_
- **Arguments:** `tak:gdy-tak nie:gdy-nie`, where `gdy-tak` and `gdy-nie` are functions
- **Returns:** (described later)

Conditionally executes `gdy-tak` without any arguments if this value is _true_, otherwise executes `gdy-nie`. Returns the value returned by the function executed.

## Number type

- **Type name:** _Liczba_
- **Literals:** number literals, as defined in the syntax specification.

The number type is defined as a IEEE 754-compliant binary64.

### `nieskończoność`

- **English name:** _infinity_
- **Arguments:** none
- **Returns:** number

Returns the numeric value representing ∞.

### `pi`

- **English name:** _pi_
- **Arguments:** none
- **Returns:** number

Returns an approximation of π.

### `przeciwna`

- **English name:** _negative_
- **Arguments:** none
- **Returns:** number

Returns the number, negated.

### `dodać:`

- **English name:** _plus:_
- **Arguments:** `dodać:x`, where `x` is a number
- **Returns:** number

Returns the number, plus `x`.

### `odjąć:`

- **English name:** _minus:_
- **Arguments:** `odjąć:x`, where `x` is a number
- **Returns:** number

Returns the number, minus `x`.

### `razy:`

- **English name:** _times:_
- **Arguments:** `razy:x`, where `x` is a number
- **Returns:** number

Returns the number, multiplied by `x`.

### `przez:`

- **English name:** _divided-by:_
- **Arguments:** `przez:x`, where `x` is a number
- **Returns:** number

Returns the number, divided by `x`.

### `równe:`

- **English name:** _equal-to:_
- **Arguments:** `równe:x`, where `x` is a number
- **Returns:** boolean

Returns whether the two numbers are equal.

### `mniejsze-od:`

- **English name:** _less-than:_
- **Arguments:** `mniejsze-od:x`, where `x` is a number
- **Returns:** boolean

Returns whether this number is less than `x`.

### `mniejsze-lub-równe:`

- **English name:** _less-than-or-equal-to:_
- **Arguments:** `mniejsze-lub-równe:x`, where `x` is a number
- **Returns:** boolean

Returns whether this number is less than or equal to `x`.

### `wartość-bezwzględna`

- **English name:** _abs_
- **Arguments:** none
- **Returns:** number

Returns the absolute value of this number.

### `mniejsza:`

- **English name:** _min:_
- **Arguments:** `mniejsza:x`, where x is a number
- **Returns:** number

Returns this number if it's less than `x`, otherwise returns `x`.

### `większa:`

- **English name:** _max:_
- **Arguments:** `większa:x`, where x is a number
- **Returns:** number

Returns this number if it's greater than `x`, otherwise returns `x`.

### `do-potęgi:`

- **English name:** _pow:_
- **Arguments:** `do-potęgi:n`, where n is a number
- **Returns:** number

Returns this number raised to the `n`th power.

### `pierwiastek`

- **English name:** _sqrt_
- **Arguments:** none
- **Returns:** number

Returns the square root of this number.

### `pierwiastek-sześcienny`

- **English name:** _cbrt_
- **Arguments:** none
- **Returns:** number

Returns the cubic root of this number.

### `zaokrąglij`

- **English name:** _round_
- **Arguments:** none
- **Returns:** number

Returns this number, rounded to the nearest integer.

### `zaokrąglij-w-dół`

- **English name:** _floor_
- **Arguments:** none
- **Returns:** number

Returns this number, rounded _downward_ to the nearest integer.

### `zaokrąglij-w-górę`

- **English name:** _ceil_
- **Arguments:** none
- **Returns:** number

Returns this number, rounded _upward_ to the nearest integer.

### `do:`

- **English name:** _up-to:_
- **Arguments:** `do:x`, where `x` is a number
- **Returns:** range

Returns the closed range from this value up to `x`.

### `razy-powtórz:`

- **English name:** _times-repeat:_
- **Arguments:** `razy-powtórz:pętla`, where `pętla` is a [loop closure](#loop-closure-type) accepting a single argument
- **Returns:** (described below)

Loops `n` times, where `n` is this number.

## Token type

- **Type name:** `Token`
- **Literal:** can be obtained in literal form by indexing literal blocks, like `'{a} element: 1`.

### `rodzaj`

- **English name:** _kind_
- **Arguments:** none
- **Returns:** string

Returns a string containing the token's kind. The string may be one of the following:
- `liczba` - number literals
- `ciąg-znaków` - string literals
- `identyfikator` - identifiers
- `(`, `)`, `{`, `}` - brackets
- `:`, `;`, `'`, `=`, `<-`, `->`, `|` - punctuation

### `jako-liczba`

- **English name:** _as-number_
- **Arguments:** none
- **Returns:** number

Returns the value of a number token. Panics if the token is not a number.

### `jako-ciąg-znaków`

- **English name:** _as-string_
- **Arguments:** none
- **Returns:** [string](#string-type)

Returns the value of a string token. Panics if the token is not a string.

### `jako-identyfikator`

- **English name:** _as-identifier_
- **Arguments:** none
- **Returns:** [string](#string-type)

Returns the value of an identifier token. Panics if the token is not an identifier.

### `prosty:`

- **English name:** _trivial:_
- **Arguments:** [string](#string-type)
- **Returns:** token

Returns a trivial token constructed from the provided kind. The kind must be one listed in `rodzaj`'s documentation, except `liczba`, `ciąg-znaków`, and `identyfikator` - for these, dedicated methods must be used.

Panics if an invalid kind is provided.

### `liczba:`

- **English name:** _number:_
- **Arguments:** number
- **Returns:** token

Returns a number token containing the given number.

### `ciąg-znaków:`

- **English name:** _string:_
- **Arguments:** [string](#string-type)
- **Returns:** token

Returns a string token containing the given string.

### `identyfikator:`

- **English name:** _identifier:_
- **Arguments:** [string](#string-type)
- **Returns:** token

Returns an identifier token containing the given identifier. Unlike in source code, any string is allowed.

## List type

- **Type name:** `Lista`
- **Literal:** string literals and literal blocks, as described in the language specification.

The list type is used for collecting values into an ordered list. For simplicity, string literals are represented as lists of numbers.

Unlike values described previously, lists are mutable; values can be modified, appended, removed, and everyone will be able to see the modifications made to the list.

### `nowa`

- **English name:** _new_
- **Arguments:** none
- **Returns:** list

Creates a new, empty list.

### `długość`

- **English name:** _length_
- **Arguments:** none
- **Returns:** number

Returns the length of the list.

### `element:`

- **English name:** _element:_
- **Arguments:** `element:indeks`, where `indeks` is a number
- **Returns:** (described below)

Returns the element at the given index. Panics if the index is out of bounds.

The first element is located at index 1.

### `dopisz:`

- **English name:** _append:_
- **Arguments:** `dopisz:element`, where `element` is any value
- **Returns:** itself

Appends an element to the list, and returns the list.

### `wymaż:`

- **English name:** _remove:_
- **Arguments:** `wymaż:indeks`, where `indeks` is a number
- **Returns:** itself

Removes the element at the given index, and returns the list.

### `elementy:`

- **English name:** _each:_
- **Arguments:** `elementy:pętla`, where `pętla` is a [loop closure](#loop-closure-type) accepting the element and its index as arguments
- **Returns:** itself

Loops over each element in the list, and passes the element and its index (in that order) as arguments to the loop closure.

### `zawiera:`

- **English name:** _has:_
- **Arguments:** `zawiera:element`, where `element` is any value
- **Returns:** boolean

Returns whether the list contains the given element.

### `znajdź:`

- **English name:** _find:_
- **Arguments:** `znajdź:element`, where `element` is any value
- **Returns:** number or nothing

Returns the index of the given element, or nothing if the element is not part of the list.

## String type

The string type is a specialization of the list type. It's a list that only contains numbers. The numbers must be valid Unicode codepoints.

## Token list type

The token list type is a specialization of the list type. It's a list that only contains tokens, and is produced by literal block literals.

## Closure type

- **Type name:** `Funkcja`
- **Literal:** closure literals, as described in the language specification.

Also known as the function type.

### `aplikuj:`

- **English name:** _apply:_
- **Arguments:** `aplikuj:argumenty`, where `argumenty` is a list of arbitrary values
- **Returns:** (described below)

Executes the closure, passing the given arguments to it. Returns the value the closure returned.

### `od:`

- **English name:** _of:_
- **Arguments:** `od:a`, where `a` is any value
- **Returns:** (described below)

Executes the closure, passing the given argument to it. Returns the value the closure returned.

Additional variants of this message are available, for passing up to 8 arguments at a time:

- `od:i:`
- `od:i:i:`
- `od:i:i:i:`
- `od:i:i:i:i:`
- `od:i:i:i:i:i:`
- `od:i:i:i:i:i:i:`
- `od:i:i:i:i:i:i:i:`
- `od:i:i:i:i:i:i:i:i:`

Used like so:
```
z = f od: x i: y;
```

When more arguments are needed, `aplikuj:` should be used.

## Loop closure type

A loop closure is a specialization of the [closure type](#closure-type). A loop closure accepts arguments defined by the loop, and returns nothing, [Break](#break-type), or [Continue](#continue-type).

When `Break` is returned, this signals the underlying loop that iteration should stop immediately. When `Continue` is returned, nothing should be done, and the loop should carry on to the next iteration.

## Loop type

- **Type name:** `Powtarzaj`

The loop type is a low-level construct allowing for implementing more high-level loops.

### `zawsze:`

- **English name:** `always:`
- **Arguments:** `zawsze:pętla`, where `pętla` is a [loop closure](#loop-closure) accepting no arguments
- **Returns:** nothing

Loops forever, until the loop closure returns [Break](#break-type).

## Import type

- **Type name:** `Importuj`

The import type is a helper object for loading external code.

### `moduł:`

- **English name:** `module:`
- **Arguments:** `moduł:nazwa`, where `nazwa` is a [string](#string-type) containing a module name
- **Returns:** (described below)

Loads the module with the given name, executes it, caches its returned value in an internal store, and returns that value. The next time `moduł:` is called with the same argument, it will return the value returned previously by the loaded module.

Module names are very much like normal file names, but the path separator is OS-independent (it's always `/`) and the `.ctk` extension should be omitted. All modules are imported relative to the current module.

An example of how to use modules:

**File:** Dodawacz.ctk
```
Dodawacz = Obiekt nowy;

Dodawacz <- nowy:wartość {
   -wartość = wartość;
}

Dodawacz <- dodaj-do:x {
   -> x dodać: -wartość;
}

-> Dodawacz
```

**File:** Program.ctk
```
Dodawacz = Importuj moduł: "Dodawacz";

dwa = Dodawacz nowy: 2;
dwa doda-do: 3 | wypisz;
```

### `plik:`

- **English name:** `file:`
- **Arguments:** `plik:nazwa`, where `nazwa` is a [string](#string-type) containing a filename relative to the current work directory
- **Returns:** closure

Loads the code in a file into a closure, and returns that closure.

For most purposes the module system (`moduł:`) should be preferred.

### `kod:`

- **English name:** `code:`
- **Arguments:** `kod:kod`, where `kod` is a [string](#string-type) containing ComesTalk code, or a [token list](#token-list-type).
- **Returns:** closure

Loads the code from a string into a closure, and returns that closure.

### `tokeny:`

- **English name:** `tokens:`
- **Arguments:** `tokeny:kod`, where `kod` is a [string](#string-type) that can be read by the ComesTalk lexer (but does not necessarily have to be valid source code).
- **Returns:** [token list](#token-list-type)

Loads tokens from source code.

# Library types

The following paragraphs describe types that may be implemented in ComesTalk code. Implementations may choose to implement these types in a lower-level language for performance reasons.

## Range type

- **Type name:** `Przedział`

The range type describes a range of numbers that can be iterated over.

### `dolna`

- **English name:** `lower`
- **Arguments:** none
- **Returns:** number

Returns the lower bound of the range.

### `górna`

- **English name:** `upper`
- **Arguments:** none
- **Returns:** number

Returns the upper bound of the range.

### `elementy:`

- **English name:** `each:`
- **Arguments:** `elementy:pętla`, where `pętla` is a [loop closure](#loop-closure-type) accepting numbers
- **Returns:** itself

Loops over all integers in the range, including both bounds.

## Break type

- **Type name:** `Wyjdź`

The Break type is a helper type used to signal that a loop should be broken out of.

## Continue type

- **Type name:** `Kontynuuj`

The Continue type is a helper type used to signal that a loop should end its current iteration immediately, and continue on with the next iteration.
