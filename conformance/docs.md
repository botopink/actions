# actions — conformance

> Path: `repository/actions/conformance/`
> Sibling (AGENTS): [`./AGENTS.md`](AGENTS.md)

`actions` is the server-action protocol both halves of an action read and write: the texts a server writes and a
browser reads, and back, in one implementation compiled for erlang and commonJS so the two sides cannot drift
(decisions 116 rule 2, 326). It is a library of its own, a dependency a program declares and imports with `from "actions"`
(326, 242), and it names no field and no header: the hidden form field and the action header are the deployment's values,
handed to both sides by whoever wires them (114 (7)). This reference describes the library as its `src/` declares it
today. Every example is a conformance cell (decisions 461, 465): the examples that exist are the decision cells of
[`modules/decisions-taken/`](modules/decisions-taken/), each a `docs.md` beside its program; the feature cells come with
front 159's Conformance step. A table, a grammar or a wire format is not a program and is fenced as `text`.

| Section | What |
|---|---|
| *Package layout* | the five modules, what the package imports, the manifest |
| *state* | the `state` grammar and `ActionState` |
| *envelope* | the v1 result envelope, its writer and its strict reader |
| *rpc* | the JSON-RPC body of a scripted call |
| *refresh* | the header value that asks for a re-render |
| *id* | the action id: one derivation, one grammar |
| *Action outcomes* | an action answers `Result`, and `ActionError` |
| *Departures from the spelling in force* | what `src/` still writes in a retired form |

| Directory | What it holds |
|---|---|
| [`modules/decisions-taken/`](modules/decisions-taken/) | The ledger's index: one `<slug>-<N>/` per code-bearing row of this repository's generated [`../decisions-taken.md`](../decisions-taken.md) (front 166, decisions 646–651), its `docs.md` walking through one example program, `draft/main.bp` until front 166 s4 promotes it to a cell (649) |

## Package layout

`src/root.bp` lists the modules `state`, `envelope`, `rpc`, `refresh` and `id`; a module is imported by its path in braces
with no `from` inside the package (`import {state.ActionState};`, 206), and by `from "actions"` from a program that
declares the dependency (`import {envelope.writeEnvelope} from "actions";`). The package is pure botopink (117 rule 8): no
host cell, no sidecar, no framework name under `src/`, so every target reads and writes the same bytes. It imports std and
`routing` (its `navigation` module, a declared dependency) and nothing else. A library runs at build on the wasm comptime
runtime (594), and `actions` has no function that needs another host to do so: a `comptime` call of any function below
answers its text at build ([libraries_run_at_build_on_wasm-594](modules/decisions-taken/libraries_run_at_build_on_wasm-594/draft/main.bp)).
The imports a program writes follow 206 and 337: a name comes from a module of its own package by the module's path
([own_module_import_without_from-206](modules/decisions-taken/own_module_import_without_from-206/draft/main.bp)), or from
a package with `from`; a bare name with neither is `shorthand-import`
([shorthand_import_refused-337](modules/decisions-taken/shorthand_import_refused-337/draft/main.bp)).

Reading is strict everywhere (decision 67): a reader answers `Result<…, string>`, and the `Error` names what was wrong,
prefixed `actions.<reader>:`. A text that is not JSON, a document that is not an object, a `v` other than `1` (a missing
`v` included) and a known key of the wrong JSON kind are each an `Error`; an unknown key is ignored.

## state

The `state` an envelope carries is a form-encoded querystring (std's `encoding.formStringify` and `formParse`,
percent-encoded). `message` is the form-level message and `f.<name>` is one field's message; any other key is ignored
when read. The writer always emits `message` first (`message=` when it is empty), then the fields in the order given:

```text
message=<message>&f.<name>=<text>&f.<name>=<text>…
```

`ActionState(ok, message, redirectTo, fields)` is what a form shows after an action. `fields` is an array of
`#(name, message)` tuples. `ok` and `redirectTo` come from the envelope's own keys, `message` and `fields` from its
`state`. Two methods read it: `fieldError(name)` answers the message of field `name`, `""` when it has none, and
`hasError()` is true when the action failed or any field carries a message.

- `newActionState(message)` is the state before any submission: `ok`, the given message, no redirect and no field messages.
- `writeState(message, fields)` writes the querystring above.
- `parseState(state)` is its inverse and answers `#(message, fields)`; a key that is neither `message` nor `f.`-prefixed is
  ignored, and `""` reads as `#("", [])`.

## envelope

The v1 result envelope is the JSON object a server action answers, with `v` first and the keys in this order:

```text
{"v":1,"ok":…,"state":"<querystring>","revalidated":[…],"redirect":"…","n":"…","payload":"…"}
```

`ActionEnvelope(ok, state, revalidated, n, payload)` carries every field but `v` and `redirect`. `n` is the navigation
signal in `routing`'s `signalToWire` form, and `redirect` is derived from it inside `writeEnvelope`: the location of a
redirect signal, `""` otherwise. It has no parameter, so no envelope can carry a `redirect` that disagrees with its signal.
The envelope's JSON is protocol only and never reaches the program's own code (303).

- `writeEnvelope(e)` answers the envelope text. JSON is written with the json library's `quote`, `array` and `object`, which
  escape every control character.
- `readEnvelope(text)` answers `Result<ActionEnvelope, string>`. Besides the strict rules above, an array key (`revalidated`)
  must hold strings only, and a `redirect` that disagrees with the location of the signal in `n` is an `Error`. A missing key
  other than `v` reads as its empty value.
- `parseActionState(envelope)` answers the `ActionState` a form shows, read from the envelope text. `ok` comes from the
  envelope's `ok` and `redirectTo` from the signal in `n`, never from `state`: a `state` carrying an `ok` key is ignored like
  any other unknown key. An envelope that does not read is a failed state whose message is the reader's error.

## rpc

The body a scripted action call sends, written by the browser and read by the server:

```text
{"v":1,"id":"<action id>","args":["…", …]}
```

`RpcCall(id, args)` holds the action id and its arguments as strings. `writeRpcBody(call)` writes the body, and
`parseRpcBody(body)` answers `Result<RpcCall, string>`. The server answers an `Error` as a 400. Each of these is an `Error`:
an empty or whitespace-only body (named as empty, not as "not JSON"), text that is not JSON, a body that is not an object, a
`v` other than `1`, an `id` that is missing, empty or not a string, and `args` that is not an array of strings. A missing
`args` is the empty list, and an unknown key is ignored.

## refresh

`refreshValue()` answers `"refresh"`, the header value that asks the server to re-render the current route with no action
(a POST whose action header carries it and no id). Both sides import it instead of spelling it twice.

## id

An action id is `a_` followed by the first 24 hex digits of `hash.hmacSha256(secret, module + "." + name + ":" + buildId)`:
stable for a build, not guessable from the name. The grammar is `a_` and exactly 24 lowercase hex digits, 26 characters:

```text
a_<24 lowercase hex digits>
```

`deriveActionId(secret, module, name, buildId)` derives the id on the server, and `isActionId(id)` checks the shape for
whoever only carries an id (a form). The secret is a parameter, since the package reads no configuration; rakun reads it
from its `rakun.actions` configuration record (324). The function is `deriveActionId`, not `actionId`, because a framework
exports an `actionId` of its own (163).

## Action outcomes

An action answers `Result`, and there is no `ActionOutcome`: a function written `#[action] pub fn signup(input: Signup, ctx:
ActionContext) -> Async<Result<Subscribed, ActionError>>` returns its value as `Ok` and `throw`s an `Error` value, and its
caller reads the same `Result<T, ActionError>` with `case` or `try`. `ActionError` is a sum type whose cases are the
outcomes, each an HTTP status on the wire: `Input(fields)` (written by the wrapper, the function never runs),
`BadRequest`, `Unauthorized`, `Forbidden`, `NotFound`, `Conflict`, `TooManyRequests` and `Internal`, each of the others
taking `(message: string)`. `ActionErrorCode` and `orError()` are gone. The decision cell
[action_answers_result-303](modules/decisions-taken/action_answers_result-303/draft/main.bp) shows an action, its caller and
a `case` over the outcomes. A `Result` is read with its own methods (`isOk()`, `map`, `unwrapOr`; there is no `result`
namespace) and a `?T` with `??`, `?.` and `!`, never methods; the cell
[optional_typescript_operators-330](modules/decisions-taken/optional_typescript_operators-330/draft/main.bp) reads a failed
action's envelope that way.

## Departures from the spelling in force

`src/` still writes these forms, and its codemod is pending; this reference describes the API in the spelling in force.

- `src/root.bp` declares its modules with `pub mod`, and `botopink.json` carries `"entry"` and `"files"`; every `.bp` under
  `src/` is a module now (592 (1)), a library has no entry (733) and `files` is the tree (703).
- `state.bp`, `envelope.bp` and `rpc.bp` write the array type as `Array<T>`, which is `T[]` (641), and import `Json`, `decode`,
  `quote`, `array` and `object` `from "std"`; the `json` library owns them since std lost its `json` module (396), and a
  consumer imports `from "json"`.
- `envelope.bp` and `rpc.bp` read the version through `Json.Num(value: f64)`; `Json` has the numbers `Int`, `BigInt` and `Dec`
  now (332 (4), 500).
- `ActionError` is decided (303) and no module of `src/` declares it yet; the decision cell's program imports it from a module
  `outcome`.
