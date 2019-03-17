# Union Types

**Example**

```ts
type StateLoading = { type: "Loading" }
type StateLoadingError = { type: "LoadingError", errorMsg: string }
type StateDisplayData = { type: "DisplayData", data: string[] }

type State = 
 | StateLoading
 | StateLoadingError
 | StateDisplayData
```

## Usage in React components

The state of React components is often growing into a complex monster of booleans and optionally available data.

Union Types can explicitly declare the stages a component can be in. The compiler enforces which data-fields are available.
Looking at the example above:
- During loading, neither `this.state.errorMsg` nor `this.state.data` is available
- When loading failed, `this.state.errorMsg` is available. `this.state.data` is not.
- When loading succeeds, `this.state.data` is available. `this.state.errorMsg` is not.

```ts
switch (this.state.type) {
    case "Loading": return <div>Loading...</div>
    case "LoadingError": return <div>Error: {this.state.errorMsg}</div>
    case "DisplayData": 
        return (
            <ul>
                { this.state.data.map(d => <li>d</li>) }
            </ul>
        )
}
```

Note: The TypeScript Compiler and IDEs fully understand the switch as a type-guard. IntelliSense helps typing out the cases.

## Exhaustive switch

**default case gets enforced for all switches**

Imagine we would want to 
- Add a state
- Remove a state

```ts
type StateLoading = { type: "Loading" }
type StateLoadingGeneralError = { type: "LoadingGeneralError", errorMsg: string }
type StateLoadingServiceUnavailable = { type: "LoadingServiceUnavailable" }
type StateDisplayData = { type: "DisplayData", data: string[] }

type State = 
 | StateLoading
 | StateLoadingGeneralError
 | StateLoadingServiceUnavailable
 | StateDisplayData
```

By just changing the type. The compiler will refuse to compile `case "LoadingError": ...`

It should also complain about:
- Not having `case "LoadingGeneralError": ...`
- Not having `case "LoadingServiceUnavailable": ...`

To achieve this:
```ts
import { ifReachable } from "./ERTS/ERTS-SafeHelpers"

switch (this.state.type) {
    case "Loading": return <div>Loading...</div>

    // COMPILER WILL COMPLAIN HERE -> "LoadingError" is not a valid state
    case "LoadingError": return <div>Error: {this.state.errorMsg}</div>
    
    case "DisplayData": 
        return (
            <ul>
                { this.state.data.map(d => <li>d</li>) }
            </ul>
        )
    
    // COMPILER WILL COMPLAIN HERE 
    // -> case "LoadingGeneralError" and case "LoadingServiceUnavailable" are not implemented
    default: throw ifReachable(this.state)
}
```

