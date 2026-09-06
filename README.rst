The Lucid language
==================

Lucid is a Python-like language sketch that keeps Python easy to read and write
while making room for cleaner type, dispatch, object, and compatibility rules.

Core principle:

    Keep Python's directness, make structure explicit, and choose the cleaner
    rule when compatibility no longer has to win.

Lucid borrows Python's readable surface syntax, Java-style single class
inheritance plus multiple interfaces, Scala-style definition-site type
information, Julia-style multiple dispatch, basedpython's fresh
per-iteration loop bindings, Kotlin-style function types, Rust's split
between recoverable and unrecoverable errors, and Swift-style toll-free
Python 3.13+ ABI bridging — made possible by a zero-deprecation release
cadence that lets Lucid choose the cleaner rule instead of the
Python-compatible one throughout the language.

Object state is declared in the class body. Construction returns fully built
objects. Public module APIs are marked with ``export``. Interfaces declare
obligations, traits provide reusable behavior, and binary operators dispatch on
both operands. Generic parameters carry definition-site variance with ``+K``,
``-K``, and ``=K``. Mutable, read-only, and immutable views are visible in the
type spelling with ``T``, ``&T``, and ``!T``.

Example
-------

.. code-block:: python

   export interface Scorable[+K]:
       def score(self, item: K) -> float

   export trait ScoreBands[+K](Scorable[K]):
       def is_confident(self, item: K) -> bool:
           return self.score(item) >= 0.8

   export class InferenceModel[=K](Scorable[K], ScoreBands[K]):
       weights: Tensor
       labels: list[K]
       scores: dict[K, float]

       factory from_checkpoint(cls, path: Path, labels: list[K]):
           weights = Tensor.load(path)
           return construct(weights, labels, {:})

       def score(self, item: K) -> float:
           if item not in self.scores:
               self.scores[item] = self.weights.dot(encode(item))
           return self.scores[item]

       getter label_count(self) -> int:
           return len(self.labels)

   def evaluate(model: &InferenceModel[str], item: str) -> float:
       return model.score(item)

   model: InferenceModel[str] = InferenceModel.from_checkpoint("model.bin", ["cat", "dog"])
   stable: !InferenceModel[str] = freeze(model)

This example shows several core language mechanics in one place:

* exported definitions are explicitly public
* interfaces require behavior with a bodyless member, no marker keyword needed
* traits provide reusable bodies
* stored fields are declared in the class body
* factories construct exact, fully initialized objects
* getters expose computed attributes without descriptors
* mutable, read-only, and immutable views are visible in annotations

Documentation
-------------

Continue with the specification documents. Nesting groups related documents
under one theme; within a theme, and across the list top to bottom, each
document builds mostly on documents already covered above it:

* `Main ideas <docs/principles.rst>`_ — the ten ideas behind the language.
* `Names, binding, and scope <docs/names.rst>`_ — binding, destructuring,
  and scope.
* Types, mutability, and annotations

  * `Type vocabulary <docs/types.rst>`_ — what a type is, and type-level
    expressions.
  * `Mutability <docs/mutability.rst>`_ — mutable, read-only, and
    immutable views.
  * `Generics <docs/generics.rst>`_ — variance, higher-kinded parameters,
    and existentials.
  * `Numeric types <docs/numeric-types.rst>`_ — exact numeric types and
    capability interfaces.

* Modern type specification

  * `Overview <docs/type-specification.rst>`_ — why interfaces, traits,
    and classes are separate.
  * `Interfaces <docs/interfaces.rst>`_ — obligations, without state or
    bodies.
  * `Traits <docs/traits.rst>`_ — reusable behavior, in place of multiple
    inheritance.
  * `Classes <docs/classes.rst>`_ — object shape, construction, and
    inheritance.

* `Multiple dispatch <docs/dispatch.rst>`_ — dispatch on both operands,
  and beyond operators.
* `Control flow and statements <docs/control-flow.rst>`_ — conditionals,
  loops, matching, and errors.
* `Strings and collections <docs/collections.rst>`_ — literals, records,
  and TypedDict shapes.
* `Indexing <docs/indexing.rst>`_ — indexing and unpacking.
* `Calls <docs/calls.rst>`_ — call syntax, partial application, and
  anonymous functions.
* Parameters and decorators

  * `Parameters and arguments <docs/parameters.rst>`_ — the anonymous
    class, and argument gathering.
  * `Decorators <docs/decorators.rst>`_ — identity-preserving ``@``, and
    decorator factories.

* `Project configuration <docs/project-configuration.rst>`_ —
  ``project.yaml`` and ``development.yaml``.
* `Modules, projects, and public APIs <docs/modules.rst>`_ — re-exports
  and lazy imports.
* `Keyword reference <docs/keywords.rst>`_ — every keyword, in one place.

These twenty documents are the source of truth for Lucid semantics.

Implementation and tooling
--------------------------

Lucid includes a compiler and toolchain written in Rust, located in
``compiler/crates/``:

* ``lucid-syntax`` — lexer, token definitions, and recursive-descent parser.
* ``lucid-checker`` — static checker verifying types, mutability views (``T``,
  ``&T``, ``!T``), interface obligations, trait bounds, and exhaustive matching.
* ``lucid-codegen`` — ahead-of-time (AOT) native code generator producing
  optimized machine code via C99/GCC with unboxed numeric registers and hardware
  call stacks.
* ``lucid-runtime`` — reference evaluation engine and dynamic dispatch runtime.
* ``lucid-cli`` — the unified driver executable providing ``build``, ``run``,
  ``check``, and ``parse`` subcommands.

Quick start
~~~~~~~~~~~

Build the release toolchain with Cargo:

.. code-block:: bash

   cargo build --release

Compile a Lucid source file to a native binary:

.. code-block:: bash

   ./target/release/lucid build program.lucid -o program
   ./program

Run a file natively with immediate execution:

.. code-block:: bash

   ./target/release/lucid run --native program.lucid

Type-check a file and report diagnostics without compiling:

.. code-block:: bash

   ./target/release/lucid check program.lucid

Benchmarks
~~~~~~~~~~

Lucid includes an automated benchmark suite adapted from the Computer Language
Benchmarks Game (Debian/Ubuntu Shootout). The suite evaluates 10 problems
against CPython 3.14, measuring recursive call overhead, memory indexing, dense
matrix products, floating-point iteration, and object trees.

To run the suite:

.. code-block:: bash

   python3 benchmarks/run_benchmarks.py

Lucid Native achieves a 13.7x geometric mean speedup over CPython 3.14 across
all benchmarks with exact numerical output equivalence. Detailed measurements
and architectural notes are in `BENCHMARKS.md <BENCHMARKS.md>`_.

Current slogan
--------------

    Classes store data. Interfaces specify obligations. Traits provide reusable behavior. Factories construct exact classes. Attribute access is structural, not magical. Exports define the public API.

