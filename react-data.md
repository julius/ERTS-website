# Data Management

Use component state whenever possible.

## Loading Data from API
- Prepare Component States*
- Start a request in the Component-constructor
- When data arrives => setState
- When an error occurs => setState

<i>\* Requests require multiple Component States. Success cases. And Error cases. 
Use Union Types to declare all possible States &rarr; [Best Practices - Union Types](best-union-types.md)</i>

```ts
import * as React from "react"
import * as t from "io-ts"
import { ERTSComponent } from "./ERTS/ERTS-React"
import { decodeJson, DecodeJsonError } from "./ERTS/ERTS-SafeHelpers"

type Props = {}

type StateLoading = { type: "Loading" }
type StateLoadingError = { type: "LoadingError", errorMsg: string }
type StateDisplayData = { type: "DisplayData", data: string[] }

type State = 
 | StateLoading
 | StateLoadingError
 | StateDisplayData

export default class ArticleList extends ERTSComponent<Props, State> {
    constructor(props: Props) {
        super(props)

        this.state = { type: "Loading" }

        fetch("/api/data")
            .then(resp => resp.json())
            .then(json => decodeJson(t.array(t.string), json))

            .then(data => {
                this.setState({ type: "DisplayData", data: data })
            })

            .catch(error => {
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
    }

    render(): JSX.Element {
        switch (this.state.type) {
            //...
        }
    }
}

```

## Shared Data
**Avoid shared data as much as possible.**

Shared data used by multiple components is a dependency complex. 
It usually ends up coupeling multiple features of your application.

To work on one feature which uses components with shared data. 
You often have to understand how other components, belonging to other features, will affect the shared data.

A good reason to use shared data: Communication between components, in different places of the DOM.

*Try to minimize the shared data as much as possible. Do NOT misuse it for caching purposes. Caching is hard. Caching should be isolated and extremely explicit.*


## Caching

Cache invalidation is a constant source of Bugs.
- Only cache if absolutely necessary
- Caches should be very explicit
- Caches should be isolated
- Cache invalidation should be very explicit
- Shared data for communication: Do not mix it up with caching

Explicit Caches help you debug Problems with Cache invalidation. 
For each cache. Your code should allow you to click on the cache set/get/invalidate functions and look at all usages across the entire project.

*If you put a lot of fragile bombs into your building. You better have a good reason. You better know where you put every single one of them!*


## Portals

**Avoid shared data for modal and notification components**

Ideally every feature of your application is isolated into a small component tree.
Some features require components in inconvenient places of the DOM.
- Modal Dialogs
- Notifications

Instead of having the trouble of making 2 component trees communicate. Which introduces a new dependency complex.

**Use React Portals.** A first-class feature of the React-Library itself. 

[https://reactjs.org/docs/portals.html](https://reactjs.org/docs/portals.html)

The end result could look something like this:
```ts
render() {
    return (
        <div className="some-feature">
            {/* ... */}

            { 
                this.state.isDialogOpen
                ? (
                    <ModalPortal>
                        <Modal.Header>Something Happened</Modal.Header>
                        <Modal.Body>Just letting you know!</Modal.Body>
                        <Modal.Footer>
                            <button onClick={() => this.setState({ isDialogOpen: false })}>
                                Thanks!
                            </button>
                        </Modal.Footer>
                    </ModalPortal>
                )
                : null 
            }
        </div>
    )
}
```



## Flux, Redux ...

To make changes and maintenance cheap
- locality of data &rarr; quickly understand components
- good typing
    - IDE helps you navigate the code &rarr; quickly understand components
    - IDE tells you about data-structures &rarr; quickly understand components
    - compiler catches bugs &rarr; less bugs

ERTS will work with any Flux library. However, not using them will pay off, in the long run.
- Maintenance will be cheaper
- Fixing Bugs will be cheaper
- Onboarding new developers will be cheaper

Not using a Flux library &rarr; One dependency less

Most Flux libraries (eg Redux) usually implement 2 anti-patterns

- Break Locality of data:
    - State is separated from the components
    - Data is used and modified by multiple components
    - Introducing a dependency complex
    - Features can often no longer be understood in isolation

- Break typing:
    - Instead of calling normal functions
    - String messages will be send around
    - Compilers and IDEs can not help you catch Bugs 
