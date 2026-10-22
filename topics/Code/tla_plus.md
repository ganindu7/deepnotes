---
layout: default
title: TLA+ and PlusCal
permalink: /topics/Code/tla_plus
parent: Development
---

## TLA+ and PlusCal
{: .no_toc}

A one-page PlusCal cheat sheet (P-syntax), the TLA+ you write inside it, how TLC reads a `.cfg`, and where to learn more.
{: .fs-6 .fw-300}

Status: Reference · Updated: {{ page.path | file_date | date_to_string }}

PlusCal is an algorithm language that **translates to TLA+**. The algorithm goes in a comment block inside a `.tla` module, the translator fills in the TLA+ below it, and TLC (the model checker) explores every reachable state looking for a violated invariant or temporal property. In VS Code (extension `tlaplus.vscode-ide`) the two commands are **TLA+: Parse module** (translate and parse) and **TLA+: Check model with TLC**; from a shell they are `java -cp tla2tools.jar pcal.trans Name.tla` and `java -cp tla2tools.jar tlc2.TLC -config Name.cfg Name.tla`.

Everything on this page was checked against TLC 1.8.0 in October 2026. The cheat sheet is the one page to keep open; the sections below it carry the detail that did not fit.

## Cheat sheet

![PlusCal and TLA+ cheat sheet]({{ '/topics/Code/tla_plus/cheat-sheet.png' | relative_url }})

[Print-ready PDF (A4 landscape)]({{ '/topics/Code/tla_plus/cheat-sheet.pdf' | relative_url }}){: .btn .btn-outline .fs-3 .mb-2 .mr-2 }
[HTML source]({{ '/topics/Code/tla_plus/cheat-sheet.html' | relative_url }}){: .btn .btn-primary .fs-3 .mb-2 .mr-2 }

## Shape of a module

```
---- MODULE Name ----                 \* the file must be Name.tla
EXTENDS Naturals, Sequences, TLC      \* TLC gives you assert / print
CONSTANTS N                           \* values come from Name.cfg
Op(x) == x + 1                        \* plain TLA+; cannot see the variables

(* --algorithm Name                   \* `--fair algorithm` for a fair single process
variables x = 0, s \in {1, 2};        \* `=`: this start value. `\in`: TLC tries each
define                                \* TLA+ that CAN see the variables and pc:
  Inv == x >= 0                       \*   invariants, properties, helpers for assert
end define;                           \* (not process-local variables: declared later)
macro Inc(v) begin v := v + 1; end macro;             \* textual substitution, no labels
process p \in 1..N                    \* a set of processes; `self` is my id
variables mine = 0;                   \* per-process variable
begin
  L1: while TRUE do ... end while;    \* first statement and every while need a label
end process;
process q = "logger" begin L: skip; end process;      \* a single named process
end algorithm; *)
\* BEGIN TRANSLATION                  \* generated: VARIABLES, Init, Next, Spec, Termination
\* END TRANSLATION                    \* never edit between these two lines
====
```

Module name, file name and the `--algorithm` name should match (the first two must).

## Statements

| Statement | Meaning |
|:--|:--|
| `x := e;` | Assign. Two assignments to the **same** variable in one step must be joined: <code>f[1] := a &#124;&#124; f[2] := b;</code> |
| `if c then ... elsif c then ... else ... end if;` | Branch. |
| `while c do ... end while;` | Loop; must carry a label. |
| `either ... or ... end either;` | Nondeterministic branch: TLC explores every arm. |
| `with v \in S do ... end with;` | Pick any element: TLC explores every choice. Also `with v = e`. No labels inside. |
| `await c;` (also `when c;`) | The whole step can happen only when `c` is true; otherwise it is *blocked*. |
| `skip;` | Do nothing. |
| `assert e;` | TLC reports an error if `e` is false (needs `EXTENDS TLC`). |
| `print e;` | Print to TLC's output. |
| `goto L;` | Jump; ends the step. |
| `call P(a); ... return;` | Procedures: `procedure P(a) variables t; begin L: ...; return; end procedure;` |

Within one step, a statement after `x := e` sees the new `x`. `=` is comparison (and the start value in `variables`); `:=` is assignment.

## Labels: where a step begins

Everything from one label to the next executes as **one atomic step**. Other processes can run only at a label. Too few labels and the model is more atomic than the real system (bugs hidden); too many and there are more interleavings than reality (more states, still sound). Put a label wherever the real system could be interrupted.

A label is **required**:

- on the first statement of the algorithm body, of each process, of each procedure;
- on every `while`;
- on the statement after a `goto`, `call` or `return`;
- on a statement that assigns a variable already assigned earlier in the step (or use `||`);
- on the statement after an `if`/`either` whose body contains a label.

A label is **forbidden** inside a `with` body and inside a macro. The translator reports every one of these, a double assignment included, as `Missing label at the following location` with a line and column.

## Invariants vs temporal properties

| | What | Checked | `.cfg` |
|:--|:--|:--|:--|
| invariant | predicate on **one state** (`x <= N`, `~(pc[0] = "cs" /\ pc[1] = "cs")`) | in every reachable state | `INVARIANTS Name` |
| temporal property | predicate on **whole behaviours** | on every infinite behaviour | `PROPERTIES Name` |

Temporal operators: `[]P` always, `<>P` eventually, `P ~> Q` leads to (whenever P, Q then or later), `[]<>P` infinitely often, `<>[]P` eventually forever. `Termination` is generated whenever the algorithm can finish (not when every process is a `while TRUE`). Liveness (`<>`, `~>`) **needs fairness**, or TLC finds a behaviour that just stops.

## Fairness

- `fair process` gives weak fairness `WF`: a step that is enabled *continuously* is eventually taken. Enough for most algorithms, including Peterson.
- `fair+ process` gives strong fairness `SF`: a step enabled *infinitely often* (even with gaps) is eventually taken. Needed when a waiter can be repeatedly beaten to a resource.
- `--fair algorithm` gives weak fairness of the whole next-state action. For a single process that is what you want; for several processes it only says *something* keeps happening, not that *each* process does, so prefer `fair process`.
- No fairness: stuttering is allowed; safety still checks fine, liveness cannot hold.

## TLA+ you will write inside PlusCal

| Kind | Syntax |
|:--|:--|
| logic | `/\` and, `\/` or, `~` not, `=>`, `<=>`, `=`, `#` (not equal), `IF c THEN a ELSE b` |
| numbers (`Naturals`, `Integers`) | `+ - *`, `\div`, `%`, `< <= > >=`, `1..N` |
| sets | `{1, 2}`, `\in`, `\notin`, `\cup`, `\cap`, `S \ T` (difference), `\subseteq`, `{x \in S : P(x)}` filter, `{e(x) : x \in S}` map, `Cardinality(S)` (`FiniteSets`), `SUBSET S` |
| quantifiers | `\A x \in S : P`, `\E x \in S : P`, `CHOOSE x \in S : P` (deterministic pick) |
| functions (arrays, maps) | <code>[i \in S &#124;-&gt; e]</code>, `f[i]`, `DOMAIN f`, `[S -> T]` (all functions). PlusCal `f[i] := e` becomes `[f EXCEPT ![i] = e]` |
| records | <code>[a &#124;-&gt; 1, b &#124;-&gt; 2]</code>, `r.a`, PlusCal `r.a := 3`, `[a : S, b : T]` (set of records) |
| sequences (`Sequences`) | `<< >>`, `<<1, 2>>`, `Append(s, e)`, `Head(s)`, `Tail(s)`, `Len(s)`, `s \o t`, `s[i]`, `SubSeq(s, m, n)`, `Seq(S)` |
| strings | `"abc"`: just values, compare with `=` |

## The `.cfg` file

TLC does not read the module alone; it reads `Name.cfg` next to it to learn which formula is the specification, which definitions are invariants or temporal properties, and what values the `CONSTANTS` take. `Spec` and `Termination` are generated by the translator. Comments start with `\*`.

```
CONSTANTS
  N = 3
  Procs = {p1, p2}          \* model values: opaque names
SPECIFICATION Spec
INVARIANTS TypeOK Safe
PROPERTIES Termination Live
CHECK_DEADLOCK FALSE        \* only if blocking forever is intended
```

## Reading TLC's answer

- `Model checking completed. No error has been found.` plus states generated / distinct / depth.
- `Invariant X is violated.` then a trace: `State 1` (initial) ... `State n` (the bad one). Follow `pc` to see where each process is.
- `Temporal properties were violated.` and the trace ends in `Back to state k` (a loop) or `Stuttering`: an infinite behaviour on which the liveness property fails.
- `Deadlock reached.` No step is enabled and not every process is `Done`.
- `The operator X is not defined`: a name in the `.cfg` has no definition in the module.
- Translator: `Missing label at the following location` (also what a double assignment produces), or a parse error with line and column.

## Common mistakes

1. **Label placement.** No label on a `while`; a label inside `with` or a macro; no label after an `if` whose body has labels. Both too few and too many labels change the model's meaning.
2. **Assigning the same variable twice in one step.** `f[a] := 1; f[b] := 2;` is rejected (reported as `Missing label`). Write `f[a] := 1 || f[b] := 2;` for a simultaneous update, or put a label between them if they really are two steps.
3. **Forgetting fairness for liveness.** Every `<>`/`~>` property fails on a stuttering trace unless the processes are `fair`.
4. **Editing the translation.** It is regenerated on every Parse. Edit the PlusCal.
5. **Definitions that mention variables above the translation.** Variables are declared *in* the translation. Put such definitions in `define` or below `END TRANSLATION`. `define` cannot see process-local variables.
6. **Unbounded state spaces.** A counter or queue that grows forever never finishes checking. Bound it with a constant (`MaxItems`) and give the constant a small value in the `.cfg`.
7. `=` is comparison (and the start value in `variables`); `:=` is assignment.
8. Module name, file name and the `--algorithm` name should match (the first two must).

## Learning path

1. **learntla.com, core track** (Hillel Wayne): [learntla.com/core](https://learntla.com/core/). PlusCal in the same P-syntax used here.
2. **The PlusCal manual** (Lamport, P-syntax): [p-manual.pdf](https://lamport.azurewebsites.net/tla/p-manual.pdf). Reference, not a tutorial: the labelling rules and statement semantics live here.
3. **The TLA+ Video Course** (Lamport): [lamport.azurewebsites.net/video](https://lamport.azurewebsites.net/video/videos.html). Pure TLA+, for when you want to understand what the translation means.
4. Books: *Practical TLA+* (Hillel Wayne, Apress 2018) for the PlusCal-first path; *Specifying Systems* (Lamport, [free PDF](https://lamport.azurewebsites.net/tla/book.html)) for the real thing.
5. More: [tlaplus/Examples](https://github.com/tlaplus/Examples) (specs to read), [docs.tlapl.us](https://docs.tlapl.us) (tooling docs), and the [TLC command-line reference](https://github.com/tlaplus/tlaplus/blob/master/general/docs/current-tools.md).

## Files in this folder

| File | What it is |
|:--|:--|
| [cheat-sheet.png]({{ '/topics/Code/tla_plus/cheat-sheet.png' | relative_url }}) | The visual reference above, A4 landscape at 2x. |
| [cheat-sheet.pdf]({{ '/topics/Code/tla_plus/cheat-sheet.pdf' | relative_url }}) | Same design, print-ready. |
| [cheat-sheet.html]({{ '/topics/Code/tla_plus/cheat-sheet.html' | relative_url }}) | Static HTML/CSS source of the design. Fixed size, not responsive. Uses the keyboard layers [design tokens]({{ '/topics/utils/keyboard_layers/design-tokens.json' | relative_url }}). |

The PNG and PDF are rendered from the HTML with headless Chrome (it needs internet for the Google Fonts):

```shell
CHROME="/Applications/Google Chrome.app/Contents/MacOS/Google Chrome"
D=topics/Code/tla_plus
"$CHROME" --headless=new --disable-gpu --hide-scrollbars --no-sandbox --virtual-time-budget=8000 \
  --window-size=1123,794 --force-device-scale-factor=2 --screenshot="$D/cheat-sheet.png" "file://$PWD/$D/cheat-sheet.html"
"$CHROME" --headless=new --disable-gpu --no-sandbox --virtual-time-budget=8000 --no-pdf-header-footer \
  --print-to-pdf="$D/cheat-sheet.pdf" "file://$PWD/$D/cheat-sheet.html"
```
