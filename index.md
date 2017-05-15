---
layout: default
---

Jane Street is built on open source.  Here you'll find the open source
libraries that we've released.  These include:

- **Core**, an industrial strength alternative to OCaml's standard
  library. It is _not_ a compatible, drop-in replacement for the
  standard library.  We've made different design decisions, and so
  code designed for the standard library needs to be adapted to use
  Core.
- **Core_extended**, a set of useful extensions to Core.  These are
  less well tested and less stable than Core proper.
- **Async**, a monadic concurrency library
- A set of syntax extensions, including **Sexplib** and **Bin_prot**,
  which extend the OCaml language itself.  These are _not_ necessary
  to use the rest of the libraries, but they are necessary for
  building them.

## Download

You can find release tarballs
[here](https://ocaml.janestreet.com/ocaml-core) and documentation
[here](https://ocaml.janestreet.com/ocaml-core/latest/doc/).

The easiest way to install our libraries is using the
[OPAM](http://opam.ocamlpro.com) package manager.

## Interested in contributing?

Great!  We're happy to work with people who want to help grow Core and
the associated libraries.  Learn more about the practicalities of
contributing [here](contributing.html).

Core is available under the Apache open-source license.

Also, if you want to contribute to this website, you can
[fork it](https://github.com/janestreet/janestreet.github.com) and
send us a pull request.

## Documentation

- The latest [API
  documentation](https://ocaml.janestreet.com/ocaml-core/latest/doc/).
- A [Hello World
  Project](https://bitbucket.org/yminsky/core-hello-world) to get you
  started on using Core.
- A set of [installation hints](installation.html).
- The [dummy's guide to Async](guide-async.html).

## Other resources

- Join the Core mailing list
  [here](https://groups.google.com/forum/#!forum/ocaml-core).
- A guide for [contributing to Core](contributing.html).
- Read the [coding standards](coding-standards.html) page to see the
  design principles behind Core.
- The [patdiff](patdiff.html) tool. An improved diff-like tool using
  Core libraries.

## Miscellaneous

- A guide to writing [performance sensitive](ocaml-perf-notes.html)
  OCaml code.
- Read our [blog](http://ocaml.janestreet.com) to learn more about how
  we approach OCaml programming.
