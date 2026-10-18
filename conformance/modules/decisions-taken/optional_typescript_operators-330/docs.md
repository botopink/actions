# Decision 330 — `?T` is read with TypeScript's operators and has no methods; `result` goes; a type declared in a type is its associated type.

The example reads a failed action's envelope into `Form.Field`s. `readEnvelope` answers a `Result`, and the program reads it with that type's own methods: `read.isOk()`, then `read.map({ e -> e.revalidated }).unwrapOr([])`. There is no `result` namespace to call instead. `Field` is declared in the body of `Form`, so it is `Form`'s associated type, written `Form.Field` in the signature of `Form.fields`, in its constructor and in the type of `fields`.

Every optional value is read with an operator, never a method. `fields?.[0]` indexes an optional array and answers a `?Form.Field`. `first?.name ?? "no field"` reads a member only when the field is there and gives a default otherwise. `first?.message` reads a member that is itself a `?string`, and the result is a `?string`, not a `??string`, because `?.` flattens. `hint(true)?.("email")` calls an optional function value, and `hint(false)?.("email") ?? "no hint"` falls back to the default because the function is `null`. `first!.name` takes the field when the program knows it is there. A `null` at that point would stop the program with the location and the expression, the same on every target. The parentheses in `(first?.message ?? "") == "" || revalidated.length > 0` are required: `??` next to `||` with no parentheses is a compile error. Writing `?.` on a value that is never `null`, for example `text?.length()`, is a compile error too.

The program prints `true`, `email`, `taken`, `no hint`, `email` and `false`.

Cells in this repository that prove it: none yet. The library's code already reads its optionals with `??` (`root.field(key) ?? Json.Str(value: "")` in `src/envelope.bp`); its feature cells come with front 159's Conformance step (decision 465), after 144 B-20's codemod commit over the library.

[draft/main.bp](draft/main.bp)
