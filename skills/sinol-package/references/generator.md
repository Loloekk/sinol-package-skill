# Generator and input verifier

The generator is `prog/<id>ingen.cpp` built on **sinol-gen** (`prog/gen.h`). The
API is documented in [sinol-gen/guide.md](sinol-gen/guide.md) (how to write an
ingen — read it before writing or porting one) and
[sinol-gen/reference.md](sinol-gen/reference.md) (every helper: random
numbers, trees, graphs, anti-hash, printing). Working examples:
[sinol-gen/examples/](sinol-gen/examples/). This file is about *what* to
generate.

## Shape of an ingen

```cpp
gen::Generator<Test> g("abc", argc, argv);    // Test: the test as a struct with operator<<
g.example("3\n1 2\n2 3\n");                   // abc0.in - the statement example
g.ocen(random_tree, 10).seed(1);              // abc1ocen.in - public test, if the user wants one
g.group(1) | gen::relabel | gen::shuffle_edges;
g.test("1\n");                                // hand-written edge case
for (int n : {10, 100, 1000}) g.test(random_tree, n).seed(100 + n);
g.group(2);
g.test(random_tree, gen::near(MAXN)).seed(2001) | gen::check(is_deep, "deep");
return g.done();
```

Why it is done this way:

- **Every test lives in the ingen**, statement examples and ocen tests too
  (`g.example`, `g.ocen`, hand-written ones as text). `in/` and `out/` are
  generated and never committed, so nothing can get lost or drift out of sync.
  There is no `sinol_static_tests` in config.yml. One example is `abc0.in`,
  several are `abc0a.in`, `abc0b.in`, … (the statement uses
  `\makecompactexample{0}` or `{0a}`, `{0b}`).
- **Every random test has its own fixed seed**, so any test can be deleted
  without changing the others (see [Seeds](#seeds)).
- **A struct for anything beyond a line of numbers**: one `operator<<` for the
  format, modifiers (`gen::relabel`, `gen::shuffle_edges`) instead of
  shuffling by hand in every function, and `check`s for intent.
- **Random arguments** (`gen::near(MAXN)`, `gen::between(a, b)`) so that not
  every maximal test has exactly `n = MAXN`.
- `./abcingen --list` maps every test to its line, arguments and size — use it
  when a test fails the verifier or kills a solution, and when reviewing.

Never use `rand()`, `random_device`, `std::shuffle` or `<random>`
distributions: their output differs between compilers, and sio2 regenerates the
tests with its own compiler when the package is uploaded. Use the test's `R`.

`testgen.hpp` (`_templates/extras/`) is still allowed inside generator
functions if an author prefers it, but the test list stays in sinol-gen.

## Seeds

Give every test that uses `R` its own `.seed(x)`, ocen tests too. Hand-written
text tests need none.

Without it the seed comes from the test's name, and deleting a test renames the
later tests of its group: `1c` becomes `1b`, gets another seed and different
content. A test aimed at a wrong solution can silently stop catching it. With
fixed seeds, deleting, inserting or moving a test only shifts names; every
other test keeps its content. `sinol-gen/guide.md` describes the library's
default; the rule here is stricter. The template's toy tests have no seeds, so
add them when replacing those tests.

- Keep seeds unique in the package. The same seed with other arguments starts
  from the same random numbers, and `gen.h` reports only identical tests.
- In a loop, derive the seed from the loop variable (`.seed(100 + n)`) and check
  that the range doesn't overlap another one.
- To re-roll one bad test, change its number.
- Draw only from the test's `R`. A generator shared between tests, or random
  numbers drawn while building the test list, make tests depend on each other
  whatever their seeds are.
- **Existing package with seeds from names:** new seed numbers change every
  random test, so `verify` has to be repeated. To keep the current tests, use
  each test's current name as its seed, **before** deleting anything:
  `.seed(gen::detail::fnv1a("1c"))`.

## Designing tests per group

For every group, within its limits:

1. **minimal** cases (n = 1, empty answer, all equal, smallest values);
2. **maximal** cases (n max, values max, answer max → overflow), several of
   them with `gen::near(MAXN)` rather than all exactly at MAXN;
3. **random** cases of a few sizes;
4. **special structures**: sorted / reverse / all equal / alternating; trees:
   path, star, caterpillar, broom, binary, deep (`trees::deep`,
   `trees::with_depth`), bounded degree; graphs: sparse, dense, disconnected (if
   allowed), many components, cycles, grids, clique chains;
5. **anti-wrong tests**: for each wrong or heuristic solution, a test where it
   fails (greedy counter-examples, worst cases for pruning, `gen::anti`
   hashing tests, a long path for recursive DFS, many equal keys);
6. **anti-slow tests**: the worst case of each slow solution, not only random
   inputs where it may be accidentally fast.

Five to twelve well-chosen tests per group beat hundreds of random ones. The
biggest test should hit every limit at once. Keep the whole package reasonably
small (≲ 100 MB of tests).

Prefer `subtask_dependencies` over copying tests into several groups.

## Checks

Use `| gen::check(pred, "what")` for what a test is *meant* to be and could
silently stop being after a typo in a parameter: maximal, without an answer,
deep, connected, the answer above 2³¹. A group-wide check
(`g.group(3) | gen::check(...)`) states a promise of that group. The limits of
the groups themselves are the input verifier's job.

## Input verifier (`prog/<id>inwer.cpp`)

sinol-make runs it with the test on stdin and the file name in `argv[1]`. It is
the only thing that checks the actual files — including the hand-written
tests and the exact format — so it must be strict.

- Read with `oi::Scanner in(stdin, oi::PL)`: `readInt(lo, hi)`, `readSpace()`,
  `readEoln()`, `readEof()`. No trailing spaces, final newline required.
- Get the group from the name (`abc2c.in` → 2, `abc0*.in` / `*ocen.in` → 0)
  and check **that group's** limits; group 0 gets the global ones.
- Check every promise of the statement: connectivity, tree, distinctness,
  sortedness, an answer exists, sum of n over test cases…
- Print one `OK ...` line with the key parameters (n, m, max value, flags):
  it shows in the `sinol-make inwer` table and makes review easy.
