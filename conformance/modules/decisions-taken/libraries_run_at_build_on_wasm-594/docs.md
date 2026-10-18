# Decision 594 — Every library runs at build on the wasm comptime runtime (434), jhonstart first; a function only Erlang or Node can run is a located error there.

The example derives an action id twice: once inside a `comptime`, which runs at build, and once at run time. `actions` is written entirely in botopink and has no host cell of its own, so every function the `comptime` reaches has a wasm form. That covers `deriveActionId`, the `hash.hmacSha256` it calls from std, and `writeState` with std's `encoding.formStringify`. The build-time calls therefore run on the wasm comptime runtime whatever `--target` the build uses, and their answers are spliced into the program as literals. The id computed at build is the one computed at run (`atBuild == atRun` prints `true` on every target), and `isActionId` accepts it. The last line writes a form state at build: the percent-encoded `message` first, then `f.email`. Nothing in the program is refused at build. If a `comptime` reached a function that only Erlang or Node can run, it would be refused at build, at the call, naming the cell, on an Erlang build as on any other.

Cells in this repository that prove it: none yet. They come with front 159's Conformance step (decision 465). Since the library has no host cell, the wasm forms it relies on are std's.

[draft/main.bp](draft/main.bp)
