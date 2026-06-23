<p align="center"><img src="https://raw.githubusercontent.com/go-embedded-ruby/brand/main/social/go-embedded-ruby.png" alt="go-embedded-ruby" width="640"></p>

<h1 align="center">go-embedded-ruby</h1>
<p align="center"><strong>A pure-Go implementation of Ruby — one static binary, full dynamism, zero cgo.</strong></p>

<p align="center">
  🌐 <a href="https://go-embedded-ruby.github.io">Website</a> ·
  📚 <a href="https://go-embedded-ruby.github.io/docs/">Documentation</a>
</p>

<p align="center">
  <a href="https://go-embedded-ruby.github.io/docs/"><img alt="Docs" src="https://img.shields.io/badge/docs-mkdocs--material-9B1C2E?style=flat-square"></a>
  <a href="https://github.com/go-embedded-ruby/ruby/blob/main/LICENSE"><img alt="License: BSD-3-Clause" src="https://img.shields.io/badge/license-BSD--3--Clause-blue?style=flat-square"></a>
  <img alt="Go 1.26.4+" src="https://img.shields.io/badge/go-1.26.4%2B-00ADD8?style=flat-square&logo=go&logoColor=white">
  <a href="https://go-embedded-ruby.github.io/docs/roadmap/"><img alt="Phases 0–7 done; Phase 8 active" src="https://img.shields.io/badge/phases-0--7%20done%20%C2%B7%208%20active-1a7f37?style=flat-square"></a>
</p>

---

go-embedded-ruby compiles Ruby to bytecode and runs it on a stack VM in the
mruby/YARV lineage, with the **lexer, parser and compiler embedded in the
binary** so `eval` and runtime `require` keep working. Ruby objects are Go heap
objects, so **Go's garbage collector is reused** — there is no GC to write.
Because it is **pure Go with cgo disabled**, it cross-compiles trivially and is
the first Ruby you can embed inside a Go program with no C toolchain
([`go-mruby`](https://github.com/mitchellh/go-mruby) needs cgo).

It is not another tree-walking interpreter: dispatch goes through **mutable
per-class method tables** (the project's `objc_msgSend`), so monkey-patching,
`define_method`, `method_missing`, singleton classes and reflection all fall out
for free. Semantics track **Ruby 4.0**, grown test-first against an **MRI 4.0.5
and JRuby** differential oracle, with **systematic performance benchmarks against
both** and validation across all six 64-bit Go targets.

## Repositories

| Repo | What it is |
|------|------------|
| [**ruby**](https://github.com/go-embedded-ruby/ruby) | the interpreter: `token`/`lexer`/`ast`/`parser`/`compiler`/`bytecode`/`vm`/`object`, plus the `rbgo` CLI |
| [**docs**](https://github.com/go-embedded-ruby/docs) | MkDocs Material documentation, versioned with [mike], served at [/docs/](https://go-embedded-ruby.github.io/docs/) |
| [**go-embedded-ruby.github.io**](https://github.com/go-embedded-ruby/go-embedded-ruby.github.io) | the Hugo landing page |
| [**brand**](https://github.com/go-embedded-ruby/brand) | logos and brand assets |

The regexp engine lives in a sibling org:
**[go-ruby-regexp/regexp](https://github.com/go-ruby-regexp/regexp)** — a pure-Go
reimplementation of Onigmo (Ruby's regexp engine), reusable beyond Ruby.

## Principles

- **Pure Go, zero cgo.** Trivial cross-compilation; a static binary by default.
- **Bytecode VM + embedded front-end.** Ruby's dynamism — `eval`, runtime
  `require`, monkey-patching, `method_missing` — stays available at runtime.
- **Reuse Go's GC.** Ruby objects are Go heap objects; nothing to collect by hand.
- **Build-time stdlib selection.** Only the libraries the `require` graph reaches
  are embedded (build tags + `//go:embed`); the Go linker drops the rest.
- **Ruby 4.0 semantics, test-driven growth.** Conformance is verified against
  **two independent Ruby 4.0 references — MRI (CRuby) 4.0.5 and JRuby** — via a
  three-way differential oracle, plus a subset of ruby/spec. The bar is real-world
  Ruby: idioms and suites from **reference applications such as Rails**
  (ActiveSupport `core_ext`) and **OpenVox/Puppet** drive the work by demand.
- **Benchmarked and portable.** Performance is measured against **MRI and JRuby**
  (pure-Go CGO=0 vs C and the JVM JIT), accelerated with [go-asmgen] where it
  helps, and validated on all six 64-bit architectures (amd64, arm64, riscv64,
  loong64, ppc64le, s390x).
- **100% test coverage** is the target, enforced as a CI gate.

## Status

**Phases 0–7 — done (core); Phase 8 (conformance & performance) active.** The
object model and essentially all of idiomatic Ruby run today, every feature
**differential-tested against MRI Ruby 4.0.5 (and JRuby)** with **100% coverage**
enforced in CI across all six 64-bit arches:

- **Values:** integers, floats, strings, symbols, arrays, ordered hashes, ranges
  (incl. beginless/endless), `Proc`/lambda, `Regexp`/`MatchData`, `Struct`.
- **Methods & blocks:** the full argument system (required/optional/`*splat`/
  keyword/`**rest`/`&block`), `{ }`/`do…end` blocks, `yield`, `block_given?`,
  `Proc`/`lambda`/stabby `->(){}`, `&proc` block-pass, `Symbol#to_proc`, setter
  defs `def name=`, endless methods `def foo = expr`, `super`.
- **Classes & modules:** inheritance, `@ivars`, `new`/`initialize`, constants and
  constant assignment, class methods `def self.foo`, `include` mixins,
  `attr_accessor`/`reader`/`writer`, `Struct.new`.
- **Metaprogramming:** `method_missing`, `send`/`public_send`, `respond_to?`,
  `define_method`, `instance_eval`/`instance_exec`,
  `class_eval`/`module_eval`/`class_exec`, `instance_variable_get`/`set`/`defined?`.
- **Control flow:** `if`/`unless`/`while`/`until` + modifiers, `case`/`when`,
  `begin`/`rescue`/`ensure`/`else`/`retry`, `break`/`next`.
- **Collections:** Array/Hash/Range with `Enumerable` and `Comparable`, both
  written once in embedded Ruby.
- **Strings:** interpolation, `%`/`format`/`sprintf`, and a broad method set.
- **Regular expressions:** `/re/imx` literals, `Regexp`/`MatchData`,
  `=~`/`match`/`scan`/`gsub`/`sub`/`split` and the match globals, on the pure-Go
  [go-ruby-regexp/regexp](https://github.com/go-ruby-regexp/regexp) engine (build stays CGO=0).

Still ahead: mutable String, pattern matching `case/in`, Fiber, bignum. The
[roadmap](https://go-embedded-ruby.github.io/docs/roadmap/) runs through Phase 8.

BSD-3-Clause.

[mike]: https://github.com/jimporter/mike
[go-asmgen]: https://github.com/go-asmgen/asmgen
