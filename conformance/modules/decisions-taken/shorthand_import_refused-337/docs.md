# Decision 337 — The shorthand import goes; `mod m;` binds the namespace `m` in the module that declares it.

The example prints a form's summary line. `main` declares `mod form;`, and that binds the namespace `form` in `main`, so `form.summary(state)` is called with no import line. `draft/form.bp` declares `summary` as `pub` and reads the state with the library's `parseState`. Every other import names where its names come from: `import {state.writeState} from "actions";` and `import {state.parseState} from "actions";` name the library, and `io.terminal` comes `from "std"`. The program prints `Check the form: email`.

The refused spelling is the shorthand. `import {summary};` has no `from`, and its first segment names no module of the package. It is refused with `shorthand-import` at the item, and because only `form` declares `summary` as `pub`, the fix is written out: `write import {form.summary};`. A bare `import {parseState};` does not reach the library either, even though only `actions` declares that name. In `main`, `import {form};` beside `mod form;` is refused with `redundant-module-import`, and the fix is to delete the item. A top-level `fn form` in `main` is refused with `import-name-collision`.

Cells in this repository that prove it: none yet. The library's `src/root.bp` declares its modules with `pub mod`, and its modules import each other by path; its feature cells come with front 159's Conformance step (decision 465).

[draft/main.bp](draft/main.bp)
[draft/form.bp](draft/form.bp)
