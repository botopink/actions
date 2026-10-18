# Decision 337 — The shorthand import goes; `mod m;` binds the namespace `m` in the module that declares it.

The example prints a form's summary line. `draft/form.bp` is a module of the package, so it declares `summary` as `pub` and reads the state with the library's `parseState`. `main` imports that function by the module's path: `import {form.summary};`, with the module first and then the item, and no `from`. Every other import names its source: `import {state.writeState} from "actions";` and `import {state.parseState} from "actions";` reach the library, and `io.terminal` comes `from "std"`. The program prints `Check the form: email`.

The refused spelling is the shorthand. `import {summary};` has no `from`, and its first segment does not name a module of the package. It is refused with `shorthand-import` at the item. Only `form` declares `summary` as `pub`, so the error gives the fix: `write import {form.summary};`, which is the line the example uses. A bare `import {parseState};` does not reach the library either, even though only `actions` declares that name; it has to be written `from "actions"`.

The decision's parts on `mod` / `pub mod` namespaces and re-export by value are overtaken by decision 592 (ctr-x): a file is a module, and there is no `mod`.

Cells in this repository that prove it: none yet. The library's modules already import each other by path (`import {state.ActionState, state.parseState};` in `src/envelope.bp`). Its feature cells come with front 159's Conformance step (decision 465).

[draft/main.bp](draft/main.bp)
[draft/form.bp](draft/form.bp)
