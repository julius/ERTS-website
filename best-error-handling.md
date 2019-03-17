# Error Handling


## The Javascript Problem

**TypeScript can not guarantee that all possible errors have been handled**

- JS allows `throw`ing anything, anywhere, at any time
- Often an `Error` object is thrown
- Syncronous code can handle errors with `try { ... } catch (error) { ... }`
- Promise code can handle errors with `.then(x => { ... }).catch(error => { ... })`
- `error` in both cases will be of type `any`


## Promises

ERTS enforces the usage of `.catch` for all Promises. 

Error cases should be handled always.


## Type Guards

Unfortunately errors are always untyped. The compiler has no way of exhaustively listing up all possible errors.

It is recommended to 
- use type-guards &rarr; to document all expected errors
- use type-guards &rarr; to have types in as many code branches as possible
- handle `error instanceof Error` or `error` being something completely unexpected

```ts
fetch("/api/data")
    .then(resp => resp.json())
    .then(json => decodeJson(SomeDataDecoder, json))

    .then(data => {
        this.setState({ type: "DisplayData", data: data })
    })

    .catch(error => {
        if (error instanceof ValidationError) {
            this.setState({ type: "DisplayValidationError", validationErrors: error.validationErrors })
            return
        }

        if (error instanceof DecodeJsonError) {
            this.setState({ type: "LoadingError", errorMsg: error.message })
            return
        }

        if (error instanceof Error) {
            this.setState({ type: "LoadingError", errorMsg: error.message })
            return
        }
        
        this.setState({ type: "LoadingError", errorMsg: "Unexpected Error" })
    })
```

## ERTSError

It is recommended to make well named Error-Subclasses for all expected Error-Cases.

Unfortunately, subclassing the normal JS Error-Class is broken in current browsers.
`error instanceof Error` will be `false` if it is a subclass of `Error`, defined in your code.

Please subclass `ERTSError` instead to avoid that JS-Bug.

```ts
export class ValidationError extends ERTSError {
    constructor(public messages: { [key: string]: string[] }) {
        super(
            "Validation Errors: \n" +
                Object.keys(messages)
                    .map((key) => key + " " + messages[key].join(" / "))
                    .join(" \n"),
        )
    }
}
```
