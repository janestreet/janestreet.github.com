---
layout: default
---

# Installation hints

The simplest way to install Core, Async, and the rest of our libraries
is to use [OPAM](http://opam.ocamlpro.com).

## Core in the toplevel

If you want to use Core and its associated libraries and syntax
extensions in the toplevel, you can put the following in your
`.ocamlinit` file:

<pre class="sh_caml">
#use "topfind"
#thread
#require "dynlink"
#camlp4o
#require "bin_prot.syntax"
#require "sexplib.syntax"
#require "variantslib.syntax"
#require "fieldslib.syntax"
#require "comparelib.syntax"
#require "core"
#require "async"
#require "core_extended"
#require "core.top"
open Core.Std
</pre>

## Building with Core and OCamlBuild

If you want to build using `ocamlbuild`, then you can put the
following in your `_tags` file:

<pre class="sh_caml">
true: syntax(camlp4o)
true: package(core,sexplib.syntax,bin_prot.syntax,comparelib.syntax,fieldslib.syntax,variantslib.syntax)
true: thread,debug
</pre>

Then you can build your app by calling:

<pre class="sh_sh">
$ ocamlbuild -use-ocamlfind myproject.native.
</pre>
