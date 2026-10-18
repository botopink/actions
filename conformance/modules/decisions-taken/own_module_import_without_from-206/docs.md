# Decision 206 — `import {…} from "<name>"` resolves only to a package — std, a bundled package or a declared dependency.

The example writes an action envelope and logs it. The package has a module of its own named `envelope` (`draft/envelope.bp`, which holds `banner`), and the `actions` library it depends on has an `envelope` module too, so one name means two things and each import line says which. `import {envelope.ActionEnvelope, envelope.writeEnvelope, state.writeState} from "actions";` names a package, so it reaches the library's `envelope` and `state` modules and never the module beside `main`, even though that module has the same name. `import {envelope.banner};` is the brace form with no `from`: the module's path, then the item, so it reaches this package's own `envelope.bp`: every `.bp` file of the package is one of its modules, named by its path, so no declaration is needed. `io.terminal` comes `from "std"`, a package like `actions`.

`main` builds an `ActionEnvelope` with the library's `writeState` and `writeEnvelope` and prints it through the package's `banner`, so the line starts with `envelope {"v":1,"ok":true,…`. Had the example written `import {envelope.banner} from "actions";`, the import would look for `banner` in the library, which does not declare it. Writing `from` with a module of this package as the source (`import {banner} from "envelope";`) is refused with `module-import-with-from` at the source string, and the fix is the brace form the example uses.

Cells in this repository that prove it: none yet. The library's own modules already import each other in the brace form (`import {state.ActionState, state.parseState};` in `src/envelope.bp`); its feature cells come with front 159's Conformance step (decision 465).

[draft/main.bp](draft/main.bp)
[draft/envelope.bp](draft/envelope.bp)
