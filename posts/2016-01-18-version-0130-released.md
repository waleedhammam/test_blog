---
title: "Version 0.13.0 released"
author: Dominik Picheta
tags: nim, python
---

Once again we are proud to announce the latest release of the Nim compiler
and related tools. This release comes just 3 months after the last
release!

A new version of Nimble which depends on this release, has also been
released. See [this](http://forum.nim-lang.org/t/1912) forum thread for
more information about the Nimble release.

This release of Nim includes over 116 bug fixes, many of which are related
to closures. The lambda lifting algorithm in the compiler has been completely
rewritten, and some changes have been made to the semantics of closures in
Nim as a result. These changes may affect backwards compatibility and are all
described in the section below.

With this release, we are one step closer to Nim version 1.0.
The 1.0 release will be a big milestone for Nim, because after that version
is released there will be no more breaking changes made to the language
or the standard library.

That being said, the next release will likely be Nim 0.14. It will focus on
improvements to the GC and concurrency. We will in particular be looking at
ways to add multi-core support to async await. Standard library improvements
are also on our roadmap but may not make it for Nim 0.14.

As always you can download the latest version of Nim from the
[download]({{site.baseurl}}/install.html) page.

Happy coding!

Changes affecting backwards compatibility
-----------------------------------------

- ``macros.newLit`` for ``bool`` now produces false/true symbols which
  actually work with the bool datatype.
- When compiling to JS: ``Node``, ``NodeType`` and ``Document`` are no longer
  defined. Use the types defined in ``dom.nim`` instead.
- The check ``x is iterator`` (used for instance in concepts) was always a
  weird special case (you could not use ``x is proc``) and was removed from
  the language.
- Top level routines cannot have the calling convention ``closure``
  anymore.
- The ``redis`` module has been moved out of the standard library. It can
  now be installed via Nimble and is located here:
  https://github.com/nim-lang/redis
- ``math.RunningStat`` and its associated procs have been moved from
  the ``math`` module to a new ``stats`` module.


## Syntax changes


The parser now considers leading whitespace in front of operators
to determine if an operator is used in prefix or infix position.
This means that finally ``echo $foo`` is parsed as people expect,
which is as ``echo($foo)``. It used to be parsed as ``(echo) $ (foo)``.

``echo $ foo`` continues to be parsed as ``(echo) $ (foo)``.

This also means that ``-1`` is always parsed as prefix operator so
code like ``0..kArraySize div 2 -1`` needs to be changed to
``0..kArraySize div 2 - 1``.

This release also adds multi-line comments to Nim. The syntax for them is:
``#[ comment here ]#``. For more details read the section of
the [manual](docs/manual.html#lexical-analysis-multiline-comments).

## Iterator changes

Implicit return type inference for iterators has been removed from the language. The following used to work:

```nim
iterator it =
  yield 7
```

This was a strange special case and has been removed. Now you need to write it like so which is consistent with procs:

```nim
iterator it: auto =
  yield 7
```

## Closure changes

The semantics of closures changed: Capturing variables that are in loops do not produce a new environment. Nim closures behave like JavaScript closures now.

The following used to work as the environment creation used to be attached to the loop body:

```nim
proc outer =
  var s: seq[proc(): int {.closure.}] = @[]
  for i in 0 ..< 30:
    let ii = i
    s.add(proc(): int = return ii*ii)
```

This behaviour has changed in 0.13.0 and now needs to be written as:

```nim
proc outer =
  var s: seq[proc(): int {.closure.}] = @[]
  for i in 0 ..< 30:
    (proc () =
      let ii = i
      s.add(proc(): int = return ii*ii))()
```

The reason is that environment creations are now only performed once
per proc call. This change is subtle and unfortunate, but:

1. Affects almost no code out there.
2. Is easier to implement and we are at a point in Nim's development process where simple+stable wins over perfect-in-theory+unstable-in-practice.
3. Implies programmers are more in control of where memory is allocated which is beneficial for a systems programming language.

Bugfixes
--------

The list below has been generated based on the commits in Nim's git
repository. As such it lists only the issues which have been closed
via a commit, for a full list see
[this link on Github](https://github.com/nim-lang/Nim/issues?utf8=%E2%9C%93&q=is%3Aissue+closed%3A%222015-10-27+..+2016-01-19%22+).

- Fixed "Generic arguments cannot be used in templates (raising undeclared identifier)"
  ([#3498](https://github.com/nim-lang/Nim/issues/3498))
- Fixed "multimethods: Error: internal error: cgmeth.genConv"
  ([#3550](https://github.com/nim-lang/Nim/issues/3550))
- Fixed "nimscript - SIGSEGV in except block"
  ([#3546](https://github.com/nim-lang/Nim/issues/3546))
- Fixed "Bool literals in macros do not work."
  ([#3541](https://github.com/nim-lang/Nim/issues/3541))
- Fixed "Docs: nativesocket.html - 404"
  ([#3582](https://github.com/nim-lang/Nim/issues/3582))
- Fixed ""not nil" return types never trigger an error or warning"
  ([#2285](https://github.com/nim-lang/Nim/issues/2285))
- Fixed "No warning or error is raised even if not nil is specified "
  ([#3222](https://github.com/nim-lang/Nim/issues/3222))
- Fixed "Incorrect fsmonitor add() filter logic"
  ([#3611](https://github.com/nim-lang/Nim/issues/3611))
- Fixed ""nimble install nimsuggest" failed"
  ([#3622](https://github.com/nim-lang/Nim/issues/3622))
- Fixed "compile time `excl ` cause SIGSEGV"
  ([#3639](https://github.com/nim-lang/Nim/issues/3639))
- Fixed "Unable to echo unsigned ints at compile-time"
  ([#2514](https://github.com/nim-lang/Nim/issues/2514))
- Fixed "Nested closure iterator produces internal error"
  ([#1725](https://github.com/nim-lang/Nim/issues/1725))
- Fixed "C Error on walkDirRec closure"
  ([#3636](https://github.com/nim-lang/Nim/issues/3636))
- Fixed "Error in generated c code"
  ([#3201](https://github.com/nim-lang/Nim/issues/3201))
- Fixed "C Compile-time error with generic proc type."
  ([#2659](https://github.com/nim-lang/Nim/issues/2659))
- Fixed "ICE dereferencing array pointer"
  ([#2240](https://github.com/nim-lang/Nim/issues/2240))
- Fixed "Lambda lifting crash"
  ([#2007](https://github.com/nim-lang/Nim/issues/2007))
- Fixed "Can't reference outer variables from a closure in an iterator"
  ([#2604](https://github.com/nim-lang/Nim/issues/2604))
- Fixed "M&S collector breaks with nested for loops."
  ([#603](https://github.com/nim-lang/Nim/issues/603))
- Fixed "Regression: bad C codegen"
  ([#3723](https://github.com/nim-lang/Nim/issues/3723))
- Fixed "JS backend - handle bool type in case statement"
  ([#3722](https://github.com/nim-lang/Nim/issues/3722))
- Fixed "linenoise compilation with cpp"
  ([#3720](https://github.com/nim-lang/Nim/issues/3720))
- Fixed "(???,???) duplicate case label"
  ([#3665](https://github.com/nim-lang/Nim/issues/3665))
- Fixed "linenoise compilation with cpp"
  ([#3720](https://github.com/nim-lang/Nim/issues/3720))
- Fixed "Update list of backward incompatibilities for Nim 0.12.0 in the main site"
  ([#3689](https://github.com/nim-lang/Nim/issues/3689))
- Fixed "Can't compile nimble with latest devel - codegen bug"
  ([#3730](https://github.com/nim-lang/Nim/issues/3730))
