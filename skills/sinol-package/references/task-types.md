# Task types

The contest repository has a skeleton of each (`_templates/{std,chk,lib,libsec,soc}`,
created by `sinol-make init <id> -t _templates <type>`) with toy examples of the
checker / library / interactor protocol. Start from it.

## std — compare with the model output

Output is compared with `out/<test>.out` ignoring whitespace differences
(oicompare). Use only when the correct output is unique.

## chk — output checker `prog/<id>chk.cpp`

Run as `<id>chk <input> <contestant_output> <model_output>`. Prints to stdout:

```
OK            or WRONG
<comment>     shown to the contestant (may be empty)
<percent>     only for OK: 100, 50, 33.3, 1/3 … (empty = 100)
```

Exit code 0. Anything else (crash, exit code > 2) is a *system error* on sio2
— the checker must never crash on any contestant output. Rules:
- Read the contestant output with `oi::Scanner(argv[2], wrong_format_callback, oi::PL)`
  so malformed output (letters, too long numbers, missing tokens, extra tokens
  at the end) becomes WRONG. `readEof()` after skipping whitespace.
- Validate everything: ranges, the claimed answer is actually achieved
  (recompute from the input), optimality compared with the model output.
- Partial points: print `OK` + percent (e.g. correct value, wrong certificate → 50).
- Compiled by sinol-make with `-Werror -Wall -Wextra -Wshadow -Wconversion`: no unused parameters, no implicit narrowing.
- Test it by hand on: correct output, other correct output, wrong value,
  invalid certificate, garbage, empty file, extra output.

## lib — library

`config.yml`:

```yaml
extra_compilation_files: ['abclib.cpp', 'abclib.h']
extra_compilation_args:
  cpp: ['abclib.cpp']
```

The solution is compiled together with `abclib.cpp` (which has `main`) and
includes `abclib.h`. Solutions implement the functions declared in the header.
- The library reads the test, calls the contestant, prints the result (compared
  with `.out`, or checked by `abcchk.cpp`).
- **Never close stdin or stdout.** sio2jail on SIO forbids `close(0)`, so
  `fclose(stdin)` gives RV on every test of every solution ("intercepted
  forbidden syscall close"). Read the whole test before calling the contestant
  instead; nothing is left on stdin then. The `lib` template (and its copy in
  `assets/`) still has `fclose(stdin)` up to contest-template PR #1 being
  merged: delete that line in every new package.
- Keep library state in an anonymous namespace / `static`, names unlikely to
  clash with contestants' globals.
- Rule violations (bad arguments, too many calls): print a message that cannot
  match the expected output and `exit(0)` → WA.
- Give contestants, in `dlazaw/` (in the template): `abclib.h` (the same as
  `prog/abclib.h`), a sample library and a sample wrong solution that shows how
  to call it. Contestants read these files to learn the interface, so keep
  them as plain as possible: **no comments**, no rule checks, no encryption,
  nothing but the calls the interface needs. A sample library of about twenty
  lines is the right size.
- **Ask the author which sample library to give.** There are two styles, both
  fine:
  - *Dialog in the terminal.* Every library function prints what it needs
    ("Podaj n: ", "Podaj wysokość pola (x, y): ") and reads the value the
    contestant types; at the end it prints the result and the number of calls.
    The contestant plays the hidden data by hand. Good when a few values are
    enough to try a solution. No test files are needed.
  - *Simple test reader.* The library reads a plain test from stdin, in a
    format given in the statement, keeps the hidden data and answers the calls
    from it the way the real library would. Good when typing values by hand
    would be tedious. Put example tests in `dlazaw/`.

    **The author has to review it.** A test reader shows contestants a test
    format and how the library answers, and that can tell them more than the
    statement does: how the hidden data is built, what kinds of cases the tests
    hold, or a property a solution could exploit. Right after writing it, ask
    the author to read `dlazaw/` (the sample library, its tests and the
    "Eksperymenty" text) and confirm that nothing there gives away how the task
    works or how it is tested. **Until the author answers, do not commit or
    push the package.** If they find something, fix it and ask again.

  Describe the chosen one in the statement's "Eksperymenty" section, with the
  compile and run commands. See package-format.md, dlazaw.
- Memory/time of the library count to the solution: keep it light, mention it
  in limits.

## libsec — library with encrypted tests (camps)

The library packages of the camps (`ret` in 03Grodek26, `szl` in 06Zakopane25,
`mat` in 11bukowina26) protect the library. Use `libsec` for a library task in a
camp repository, and plain `lib` otherwise, unless the user decides differently.
If the repository has no `_templates/libsec` (older contest-template), copy it
from contest-template or build the same thing from `lib`.

What the template does:

- **Guard and encryption.** The test starts with a guard word, and every number
  is written as `value ^ KLUCZ`. A solution that reads stdin before the library
  (a constructor of a global variable runs before `main`) gets only encrypted
  numbers, and the library then reports a communication error.
- **Coded verdicts.** The library prints random words, `KOD_OK <answer>` or one
  code per kind of error, never a plain verdict. `prog/abcchk.cpp` maps the codes
  to messages, and any other output is "Błąd związany z komunikacją". Output that
  starts with `OK` is "Zakaz szponcenia!!!". The out files hold the model's code
  line, so the checker compares answers, not the text.
- **Weak `main()`** and a weak contestant function in the library, so the library
  can also be linked into the ingen, the inwer and the checker.

What the template cannot do, so it is your job:

1. **New secrets for every package.** Draw them, never copy them from another
   package (whoever saw that package would know them):

   ```sh
   python3 -c "import secrets,string; w=lambda: ''.join(secrets.choice(string.ascii_lowercase) for _ in range(18)); print('STRAZNIK', w(), 'KLUCZ', secrets.randbits(31)); [print(k, w()) for k in ('KOD_OK', 'KOD_NARUSZENIE', 'KOD_KOMUNIKACJA')]"
   ```

   Put them on the `TEMPLATE:` lines of `abclib.cpp`, `abcchk.cpp`, `abcingen.cpp`
   and `abcinwer.cpp`, and check that the four copies are equal (`grep` every
   value). Add a code for every kind of rule the statement has (too many calls,
   an argument out of range, ...), each with its own message in the checker.
2. **Secrets only in those four files.** Never in a header or in another file of
   `extra_compilation_files`: those files lie next to the solution when it is
   compiled, so a solution could `#include` them.
3. **Everything through the encryption.** Every test of the ingen, the example
   and hand-written ones too, goes through `zaszyfruj` (hand-written: the `text`
   helper with plain text). Negative numbers work. The inwer checks the guard
   first and reads every number through `rd`.
4. **Statement without `\makecompactexample`.** It would print the encrypted
   test. Show the example as a table of calls ("Przykładowy przebieg programu").
   The template has the camp layout: Komunikacja, Założenia, Ocenianie,
   Przykładowy przebieg programu, Podzadania, Biblioteka, Eksperymenty. In
   "Biblioteka" put the measured time and memory of the library: a solution
   that only returns, and one that makes the maximal number of calls, on the
   largest tests. Give these numbers with a margin. Compile commands use
   `-std=c++23`.
5. **Plain files for contestants.** The sample library in `dlazaw/` knows
   nothing about the guard, the key or the verdict codes. In the test-reader
   style it reads plain tests and `dlazaw/` gets a plain `abc0.in`; in the
   terminal style there are no test files. `dlazaw.zip` contains only
   `dlazaw/`, never the encrypted tests from `in/`.

Checks before calling it done:

- Decrypt the generated tests with a scratchpad script and compare them with
  what the ingen means to write. When converting an existing package, compare
  them with its old plain tests byte by byte before replacing the tests.
- Write three cheating solutions in the scratchpad, not in the package: one
  reads stdin in a global constructor, one prints `OK` and exits, one breaks a
  rule. Each must get its own message from the checker.
- `sinol-make run`: the expected scores are unchanged after the conversion. The
  encrypted tests are about a third larger, so watch the package size.

## soc — interactive (`prog/<id>soc.cpp`)

sio2/sinol-make run the interactor as

```
abcsoc <num_processes> <read_fd_0> <write_fd_0> [<read_fd_1> <write_fd_1> …]
```

- stdin of the interactor = the test file (the solution never sees it),
- stdout of the interactor = verdict in the checker format (above),
- it talks to solution *i* through `fdopen(read_fd_i, "r")` / `fdopen(write_fd_i, "w")`;
- each solution gets its index as `argv[1]` and talks through its own stdin/stdout;
- interactor limits: 2 × the solution's time limit.

Rules:
- `signal(SIGPIPE, SIG_IGN)` and treat EOF / failed reads as WRONG
  ("program ended before …"), never crash.
- `fflush` after every message; the statement must tell contestants to flush
  (`cout << endl`, `fflush(stdout)`). The classic contestant mistake is
  `cin.tie(nullptr)` without flushing → deadlock → TLE; keep such a wrong
  solution in the package.
- Count queries and enforce limits; validate every query (range, format).
- Adaptive interactors: be sure the answer stays consistent with all previous
  replies; describe adaptivity in the statement.
- No `.out` files. Tests contain whatever the interactor needs (hidden data).
- `num_processes: k` in config for k solution copies.
- sinol-make has no `chkwer` for interactive tasks.
- Statement: no `\makecompactexample`; show an example dialogue in a table.
- Give contestants a sample interactor in `dlazaw/` (in the template): it
  talks through stdin/stdout, takes the hidden data from argv or a file,
  checks little; the statement says how to run it (`mkfifo`). See
  package-format.md, dlazaw.
