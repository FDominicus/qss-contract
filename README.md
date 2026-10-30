# qss-contract — contracts with blame for Emacs Lisp, after Racket

A contract says what a function takes and what it returns. When it is
broken, the message says **who** broke it — the caller, or the function
itself:

```elisp
(require 'qss-contract)
(qss-define-contract-type 'my-weight '(and/c numberp (lambda (w) (>= w 0))))
(qss-define-contract my-tare (-> my-weight my-weight my-weight))

(my-tare 10 -1)
;; Contract violation: my-tare: argument 2 must satisfy my-weight, got -1
;;   blame: the caller (outside the checked functions)
;;   contract: (-> my-weight my-weight my-weight)
```

- An argument that breaks the contract is the fault of the **caller**; if
  that is a checked function itself, it is named.
- A result that breaks it is the fault of the **function** — for a
  generic function, of the method for the type of the first argument.

The contract is an `:around` advice. It stands beside the code, not in
it, as `provide/contract` in Racket: one file can hold the whole checked
interface of a package. It holds for every method of a generic function,
also methods added later, and survives redefinition. `C-h f` shows it.
`qss-contract-enabled` turns all checks off (a checked function then
costs one extra call); `qss-contract-remove` removes one.

## The language

| Contract | Means |
|---|---|
| `stringp`, `natnump` … | a predicate |
| `any` | anything |
| `(or/c C ...)`, `(and/c C ...)`, `(not/c C)` | one of, all of, not |
| `(one-of V ...)` | one of these values (compared with `equal`) |
| `(listof C)`, `(list/c C ...)`, `(cons/c A D)` | lists and conses |
| `(lambda (x) ...)` | a predicate written in place |
| `(-> ARG ... RESULT)` | a function; `(opt C)` an optional argument (nil always allowed, as nil means "not given"), `(rest C)` the rest |
| `my-weight` … | a contract named with `qss-define-contract-type` |

A function passed as an argument is only checked to be a function, not
wrapped (Racket does that; this library does not, yet).

Put contracts on the boundaries of a package, not on functions that run
once per element of a loop: there they cost nothing noticeable.

## Used by

[qss](https://github.com/FDominicus/qss) checks its whole public interface
with it (`qss-interface.el`, about fifty functions); `(contracts)` there
lists them as a table.

## Tests

```
emacs --batch -L . -l qss-contract-test -f ert-run-tests-batch-and-exit
```

## License

MIT, see LICENSE.
