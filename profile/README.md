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
  <a href="https://go-embedded-ruby.github.io/docs/roadmap/"><img alt="Phase 1" src="https://img.shields.io/badge/phase-0%2B1%20object%20model-1a7f37?style=flat-square"></a>
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
for free. Semantics track **Ruby 3.4**, grown test-first against an MRI
differential oracle, with **systematic performance benchmarks** and validation
across all six 64-bit Go targets.

## Repositories

| Repo | What it is |
|------|------------|
| [**ruby**](https://github.com/go-embedded-ruby/ruby) | the interpreter: `token`/`lexer`/`ast`/`parser`/`compiler`/`bytecode`/`vm`/`object`, plus the `rbgo` CLI |
| [**docs**](https://github.com/go-embedded-ruby/docs) | MkDocs Material documentation, versioned with [mike], served at [/docs/](https://go-embedded-ruby.github.io/docs/) |
| [**go-embedded-ruby.github.io**](https://github.com/go-embedded-ruby/go-embedded-ruby.github.io) | the Hugo landing page |
| [**brand**](https://github.com/go-embedded-ruby/brand) | logos and brand assets |

The regexp engine lives in a sibling org:
**[go-onigmo/regexp](https://github.com/go-onigmo/regexp)** — a pure-Go
reimplementation of Onigmo (Ruby's regexp engine), reusable beyond Ruby.

## Principles

- **Pure Go, zero cgo.** Trivial cross-compilation; a static binary by default.
- **Bytecode VM + embedded front-end.** Ruby's dynamism — `eval`, runtime
  `require`, monkey-patching, `method_missing` — stays available at runtime.
- **Reuse Go's GC.** Ruby objects are Go heap objects; nothing to collect by hand.
- **Build-time stdlib selection.** Only the libraries the `require` graph reaches
  are embedded (build tags + `//go:embed`); the Go linker drops the rest.
- **Ruby 3.4 semantics, test-driven growth.** Conformance verified against MRI
  and a subset of ruby/spec.
- **Benchmarked and portable.** Performance is measured against reference Ruby,
  accelerated with [go-asmgen] where it helps, and validated on all six 64-bit
  architectures (amd64, arm64, riscv64, loong64, ppc64le, s390x).
- **100% test coverage** is the target, enforced as a CI gate.

## Status

**Phases 0 and 1 — done.** Phase 0: the full `source → lexer → parser → AST →
compiler → bytecode → VM` chain (integers/floats/strings, locals, arithmetic
with Ruby floor division, `if`/`unless`/`while`/`until` + modifiers, `def` +
recursion, `puts`/`print`/`p`; `puts 1 + 2` and `fib(20)` run end to end). Phase
1: the live object model — classes with inheritance, `@ivars`,
`new`/`initialize`, constants, dynamic dispatch via mutable method tables,
`method_missing`, **modules + `include` (mixins), and `super`**. Behaviour
differential-tested against MRI; 100% coverage; CI green across 6 arches. Next:
blocks & `yield`, then Phase 2 (Symbols, real String/Array/Hash). The
[roadmap](https://go-embedded-ruby.github.io/docs/roadmap/) runs through Phase 8.

BSD-3-Clause.

[mike]: https://github.com/jimporter/mike
[go-asmgen]: https://github.com/go-asmgen/asmgen
