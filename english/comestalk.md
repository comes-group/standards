# ComesTalk

| | |
| --- | --- |
| **Name** | ComesTalk Specification |
| **Version** | 2021.1 pre-release |
| **Authors** | liquidev |

Smalltalk. In Polish. Do dupy.

## Source encoding

Source code for ComesTalk shall be encoded as UTF-8, with line endings being either LF (ASCII 0Ah), or CR (ASCII 0Dh) followed by LF. Any other combinations shall be treated as an error.

## Lexis

ComesTalk uses a limited set of lexemes, as described below. Below is a glossary of all supported lexemes.
```
123 456.789 -273.15                # numbers
"abc"                              # strings
test zażółć-gęślą-jaźń Test -test  # identifiers
() {}                              # brackets
: ; ' = <- -> |                    # punctuation
```

### Whitespace and comments

Both whitespace and comments shall be ignored. Whitespace includes ASCII 20h (SPACE) and ASCII 09h (horizontal tab).

A comment starts with a `#` (23h) and ends with a line ending as described in [Source encoding](#source-encoding).

Whitespace and comments may only be ignored on token boundaries; this means that they are preserved within string literals.

### Numbers

Numbers in ComesTalk are represented by an optional negative sign `-` (2Dh), and one or more decimal digits, optionally followed by a decimal separator and more decimal digits.

Decimal digits are specified as the ASCII characters in range `0`-`9` (30h-39h). The decimal separator is the ASCII character `.` (2Eh).

The decimal separator must be followed by at least one digit and must not be the first character of the number.

### Strings

A string literal begins with the quote character `"` (22h) and follows until that same quote character is found. If that quote is followed by yet another quote, the next character is interpreted to be a quote and does not terminate the string. Otherwise the string is terminated.

Example:
```
"hello ""world""!"
```
encodes the following string:
```
hello "world"!
```

Any byte is permitted in string literals, including line breaks, and sequences that form invalid UTF-8. This allows for compact encoding of binary data - only ASCII 22h must be escaped by doubling it.

### Identifiers

Identifiers in ComesTalk start with a letter or a dash `-` (2Dh), followed by zero or more letters, dashes, or digits.

Letters include the ASCII range `A`-`Z` (41h-5Ah), `a`-`z` (61h-7Ah), as well as _any_ Unicode codepoints outside of the ASCII range. Note that grapheme clusters do not undergo any normalization, so `ź` (U+017A) and `ź` (U+007A followed by U+0301) are different identifiers! Code should avoid using combining marks and instead use dedicated codepoints for letters with diacritics instead. The standard library must follow this convention.

### Brackets

Brackets are used as code block delimiters. The two supported pairs of brackets are `()` (ASCII 28h, 29h), and `{}` (7Bh, 7Dh), used for expression grouping and blocks respectively; `[]` is not used in the current revision of ComesTalk.

### Punctuation

ComesTalk uses the following characters for punctuation:

| Character | ASCII | Usage |
| --- | --- | --- |
| `:` | 3Ah | Delimiting argument names from their values in messages. |
| `;` | 3Bh | Delimiting statements in code blocks. |
| `'` | 27h | Literal blocks. |
| `=` | 3Dh | Variable assignment. |
| `<-` | 3Ch 2Dh | Method declaration. |
| `->` | 2Dh 3Eh | Returning values. |
| <code>\|</code> | 7Ch | Chaining messages. |

## Syntax

Glossary:
```
# Expressions
1; 3.14; "abc"              # literals
x                           # variables
x m                         # unary messages
x m n o                     # chaining unary messages
x m: y                      # messages with arguments
x m: y n: z                 # messages with more than one argument
(x dodać: 1)                # grouping
{:arg: -> arg dodać: 1}     # closures
'{1 2 3}                    # literal blocks

# Statements
x;                          # expression whose value is discarded
x = y;                      # assignment
X <- m {}                   # messages without arguments
X <- m:x n:y {}             # messages with arguments
-> x                        # respond
```

### Expressions

Expressions are syntactic constructions that return a value on the semantic level.

#### Literals

A number literal or string literal is interpreted as representing its own value and does not undergo any extra processing.

#### Variables

A lone identifier is interpreted as a reference to a variable within the current environment and does not undergo any extra processing.

#### Unary messages

An expression followed by an identifier is a _unary message_ – that is, a message sent to the object on the left-hand side, without any extra arguments.

Unary messages may be chained freely.

#### Multi-argument messages

Multi-argument messages are composed of the receiver (or left-hand side), the message name, followed by a colon, followed an expression, followed by zero or more of such argument-value pairs.

Examples:
```
3 dodać: 5;  # send the message `dodać:` to 3, with the argument 5
Prostokąt szerokość: 30 wysokość: 50  # send the message szerokość:wysokość: to Prostokąt
```

Note that multi-argument messages _do not chain_ like unary messages do. For that, parentheses must be used.
```
(3 dodać: 5) dodać: 3;
(3 dodać: 5) silnia wypisz;
```
To make this common use case less painful, the `|` message piping syntax may be used.
```
3 dodać: 5 | dodać: 6 | dodać: 7;
```

#### Closures

Closures group blocks of code with optionally provided arguments.

If the opening bracket `{` is immediately followed by a colon `:`, zero or more identifiers specifying argument names shall be read. Arguments are read until another colon is reached, at which point a list of statements is read, until the closing bracket `}` is reached.

Examples:
```
# A closure accepting no arguments, returning no value.
{
   a = 3 dodać: 5;
   a wypisz;
}
# A closure accepting an argument x and returning that argument with 2 added.
{:x: -> x dodać: 2}
# A closure accepting arguments x, y, z, and returning them multipled with each other.
{:x y z: -> x razy: y | razy: z}
```

#### Literal blocks

Literal blocks collect tokens for later use via reflection. This can be used for creating syntax that would not normally be possible in ComesTalk.
```
# A C-like function declaration stored in a block, as individual tokens.
'{
   # Note that brackets may nest within literal blocks.
   int main(void)
   {
      printf("Hello, world!\n");
   }
};
```

### Statements

Statements make up a program and comprise of bare expressions, as well as declarations.

#### Statement lists

A statement list is zero or more statements. A ComesTalk program is a statement list.

#### Expression statements

An expression statement executes an expression and discards its value. It's denoted by an expression followed by a semicolon `;`.

#### Assignments

Assignments are identifiers, followed by the equals symbol `=`, followed by an expression for the variable's value, followed by a semicolon `;`.

```
a = 1;
xyz = Wektor3D x: 1 y: 2 z: 3;
```

#### Message declarations

Message declarations are identifiers, followed by `<-`, followed by a `message-name` or one or more `parameter-name:parameter-variable` pairs, followed by a block delimited by curly brackets `{}`.

```
# Create a printer for a custom Vec3 type.
Wektor3D <- wypisz {
   "(%, %, %)" formatuj: -x oraz: -y oraz: -z | wypisz;
}

# Create a pow (do-potęgi:) message.
Liczba <- do-potęgi:n {
   x = --ja;
   (1 do: n) powtarzaj: {
      x = x * n;
   };
   -> x
}
```

Note that message declarations do not end with a semicolon.

#### Response values

A message can respond with another value. A response statement must be the last statement in a message's block or closure, and is written down as `->` followed by an expression with the value.

```
# Returns 1 from the currently running program and stops execution.
-> 1
```

## Semantics

### Environments

Every running closure or message has an environment attached. This environment stores a few important things:
- the current receiver (`--ja`)
- pointers to outer variables
- variables in the current block

The following sections describe variables available in the (implicit) program block.

#### Built-in variables

The following variables are built-in and cannot be written to.

- `nic` - null value, singleton
- `tak` - true boolean value, singleton
- `nie` - false boolean value, singleton
- `--ja` - the receiver of the current message, or `nic` if not in a message

#### Built-in objects

A complete list of built-in classes and the messages they accept can be found in the [Core Specification](comestalk-core.md).

### Messages

Messages get their names from what arguments they accept. The name of a unary message is simply the message name as declared:
```
X <- test {}  # name: test
```
The names of n-ary messages are derived from the names of their parameters, with each name followed by `:`.
```
X <- m:x n:y {}  # name: m:n:
```

This overloading mechanism allows for two messages to have the same initial name, as long as their argument names or arities differ.
```
Płótno <- rysuj:co x:x y:y szerokość:s wysokość:w {}  # rysuj:x:y:szerokość:wysokość:
Płótno <- rysuj:co x:x y:y {}                         # rysuj:x:y:
```

The built-in `wypisz` message on the Object type makes use of this, allowing one to specify a writer that should write down the object.
```
w = Lista nowa;
1 wypisz;
1 wypisz: w;
```
Usually the most specific overload should be overwritten in classes inheriting from another one, eg. `wypisz:` should be overwritten instead of `wypisz`.

When a message is called on an object type, the receiver is implicitly created and assigned the appropriate type. For example:

```
Wektor = Obiekt nowy;
Wektor <- x:x y:y {
   -x = x;
   -y = y;
}

u = Wektor x: 10 y: 20;
```

In this example, `--ja` in `Wektor`'s `x:y:` will not be `Wektor` itself, but rather a new, empty object, whose type is `Wektor`.

The type of a value decides on what messages can be called on it. For more information on what messages are defined for which types, refer to the [Core Library Specification](comestalk-core.md) and the host application's documentation.

### Assignment

An assignment writes to a variable inside the current environment.
```
a = 1;     # a is then saved in the environment and can be referenced later…
a wypisz;  # like so.
```

Variable names beginning with a single dash `-` are special, because they will be written to the current receiver's _field store_ instead.
```
Kot = Obiekt nowy;

Kot <- o-imieniu:imię {
   -imię = imię;
}

Kot <- imię {
   -> -imię
}

mruczek = Kot o-imieniu: "Mruczek";
mruczek imię wypisz;
```
As shown in the example above, variables in the field store persist across different message calls.

### Closures

Closures can be created using the syntax described previously, and may be called using the built-in `wywołaj-z:` group of methods (up to 16 arguments), or the `argument:` builder, which supports an indefinite amount of arguments.
```
x = {:x y: -> x + y};

# Call using wywołaj-z:oraz:
x wywołaj-z: 1 oraz: 2 | wypisz;

# Call using argument:
x argument: 1 | argument: 2 | wywołaj wypisz;
```

Using `argument:` allows for _currying_, as it creates a new lightweight "Argument Proxy" each time it is called, which will call the destination closure once `wywołaj` is called on it.

#### Returning

The `->` response statement will stop executing the current _catching_ block. A catching block is a closure with an argument list, or a block in a message declaration. Non-catching blocks will bubble up returns to the next catching block up the stack. This allows for returning from conditionals and simple blocks alike:
```
X <- decyduj:x {
   x
      tak: {-> 2}
      nie: {-> 3};
}
X decyduj: (2 równe: 3) | wypisz;  # outputs 3
```
If catching blocks are passed to conditionals though, the conditional will respond with the value returned from the block.
```
n = x
   tak: {:: -> 2}
   nie: {:: -> 3};
n wypisz;
```
This convention of returning values from catching blocks is followed across the standard library.
