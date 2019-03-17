# Unbreakable States

Union Types for Component-State 
- Better Typing &rarr; Less Bugs
- Less Guesswork &rarr; Cheap maintenance


## Union Types vs Booleans

Components usually have multiple stages (eg: Loading Data &rarr; Display Data &rarr; Display Error).

ERTS recommends using Union Types to declare the stages explicitly.

See also [Best Practices - Union Types](best-union-types.md)




## Example
Consider a component to edit blog posts. With the following stages:
- (1) Loading: Load blog post from API
- (2) Editor: Form to edit blog post title+text
- (3) Saving: Send edited post to API
- (4) Success: Display success message

And some extra stages:
- (1.1) API-Error: Display error if loading failed
- (3.1) Editor: Handle Validation errors (eg: empty title)
- (3.2) API-Error: Display error if saving failed

**Bad Solution (Booleans)**
```ts
type State = {
    isLoading: boolean
    loadingError?: string
    form?: {
        title: string
        text: string
    }
    isSending: boolean
    validationErrors?: string[]
    sendingError?: string
    isSuccess: boolean
}
```

**Good Solution (Union Types)**
```ts
interface Form {
    title: string
    text: string
}

type StateLoading = 
    { type: "Loading" }
type StateLoadingError = 
    { type: "LoadingError", errorMsg: string }
type StateEditor = 
    { type: "Editor", form: Form }
type StateEditorSaving = 
    { type: "EditorSaving", form: Form }
type StateEditorSavingError = 
    { type: "EditorSavingError", form: Form, errorMsg: string }
type StateEditorValidation = 
    { type: "EditorValidation", form: Form, validationErrors: string[] }
type StateSuccess = 
    { type: "Success" }

type State = 
 | StateLoading
 | StateLoadingError
 | StateEditor
 | StateEditorSaving
 | StateEditorValidation
 | StateSuccess
```

## No invalid states
Using union types. The compiler will prevent the component from entering an invalid state.

With the boolean solution. If a developer makes a mistake. 
The component could enter a state where, for example, `isLoading` and `isSending` are both `true`.

## render() with switch
**With the boolean solution**

render() code usually has to do a lot of detective work. 
To figure out the stage, the component is in right now.

Often render() grows to work like this:

```ts
render() {
    if (this.state.isLoading) { 
        return  <div>Loading...</div>
    }
    // ...

    if (this.state.validationErrors && !this.state.isSending && this.state.form) {
        // ...
    }

    // ...
}
```

**With union types**

render() has to handle a list. Developers are humans. Humans are really good with flat lists.

```ts
render() {
    switch (this.state.type) {
        case "Loading": 
            return <div>Loading...</div>
        case "LoadingError":
            return this.renderLoadingError(this.state.errorMsg)
        case "Editor":
            return this.renderEditor(this.state.form)
        // ...
    }
}

renderEditor(form: Form) {
    //...
}
```

**Note**
- The switch()-statement acts as a type-guard. The TypeScript Compiler will only allow access to `this.state.errorMsg` after the type has been checked (with if/switch).
- When typing the case statements. IntelliSense will help.
- Invalid case statements will be prevented by the compiler.
- Forgotten cases can be caught by the compiler as well &rarr; See [Union Types - Exhaustive switch](best-union-types.md?id=exhaustive-switch)

## helper (() => switch { ... })()

Sometimes a switch inside of a JSX-Construct is handy.

```ts
render() {
    return (
        <section className="foo">
            <div className="container">
                {(() => 
                    switch (this.state.type) {
                        case "Loading": 
                            return <div>Loading...</div>
                        case "LoadingError":
                            return this.renderLoadingError(this.state.errorMsg)
                        // ...
                    }
                )()}
            </div>
        </section>
    )
}
```
