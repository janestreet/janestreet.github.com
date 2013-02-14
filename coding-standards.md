---
layout: default
---

# Core Principles

This document is a _brief_ overview of Core's coding standards.  For a
longer and more detailed discussion of the motivation behind these
standards, you can read this
[blog post](https://ocaml.janestreet.com/?q=node/28).

Note that Core is designed differently from INRIA's standard library,
and so Core is not a drop-in replacement.  You will have to modify
code that's designed against INRIA's stdlib to use Core.

## Types and Modules

Almost all types should have the type-name `t`, and should be kept in
a dedicated module.  This even extends to the built in types like
`int`, `float` and `bool`, which have their own modules in `Core`.

## `t` comes first

If you have a function in a module `M` that takes an argument of type
`M.t`, then that argument goes first by default.

### Error handling: exceptions, options and function names

In most circumstances, a function should not routinely throw an
exception unless it indicates so explicitly, either in the function
name, by ending with `_exn`, or in the type of the return value,
 _e.g._, using an option.  Thus, `List.find` returns an
option rather than throwing `Not_found`, and `List.find_exn` throws an
exception.

Catching of individual exceptions is discouraged, and in many cases,
individual exceptions are not exported from the modules in which they
are defined.

When using the type of the return value to model the error, consider
the following forms:

- Use `option`, when no information needs to be returned in
  the error case
- Use `'a Or_error.t`, when there is textual information to be
  returned in the error case.  (Note that this is the same as a
  `('a,Error.t) Result.t`.)
- Use a `Result.t` with a custom error type (often a polymorphic
  value) when specific error information needs to be handled
  programmatically.

Other forms (for example, an `('a,exn) Result.t`) do come up in Core,
but are not the preferred mode.

## interface includes

There are a number of standardized interfaces that we use as
components of lots of different signatures.  Thus, if you had a module
representing a type that could be converted back and forth to floats,
supported comparison and has its own hash function, you could write
the interface as follows:

<pre class="sh_caml">
module M : sig
  type t
  include Floatable  with type t := t
  include Comparable with type t := t
  include Hashable   with type t := t
end
</pre>
