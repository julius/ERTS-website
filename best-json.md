# JSON

![JSON.parse](readme/readme_example1.jpg)

## Example using io-ts

**Not Allowed**

```ts
// Less code -> Lots of bug tickets later on
fetch("/some/api")
    .then(resp => resp.json())
    .then(json => {
        const user = JSON.parse(json)
        alert("User-Email: " + user.email)
    })
```

**Good**

```ts
import * as t from "io-ts"
import { decodeJson, DecodeJsonError } from "./ERTS/ERTS-SafeHelpers"

// DECODER - DATA SCHEMA
const decoderUser = t.type({
    email: t.string,
    username: t.string,
    image: t.union([t.string, t.null]),
    bio: t.union([t.string, t.null]),
})

// TYPINGS - CREATED FROM DECODER
type User = t.TypeOf<typeof decoderUser>

// DATA FROM API
fetch("/some/api")
    .then(resp => resp.json())
    .then(json => decodeJson(decoderUser, json))
    .then(user => {
        /* guaranteed type -> user: User */
        alert("User-Email: " + user.email)
    })
    .catch(error => {
        if (error instanceof DecodeJsonError) { /* handle schema mismatch error */ }
    })
```

**Notes**

In the first example. The `user` object has the type `any`.
The structure of `user` is completely unknown. 
- API changes will not get discovered
- Developers have to look up the data-structure
- Many bug tickets will be the measurable result

In the second example. 
- The `user` object is typed
- The response gets checked to expose API changes early
- Compiler enforces handling of unexpected API responses
- The compiler guarantees: `user.email` is a string and is used in a valid way

```ts
user: { 
    email: string
    username: string
    image: string | null
    bio: string | null 
}
```

## The Problem: Untyped Data

**Developer guessing data-structures &rarr; Bugs**

Typically in JS-Frontend-Applications
- Data from an API is not verified
- Schema is in some backend documentation
- Data-Objects move through multiple components
- Components mutate Data-Objects for convenience

A developer maintaining a certain component
- Has to guess the structure of data-objects
- Will look up backend documentation (often outdated)
- Check backend code. To verify his assumptions.
- Check Data-Flow. Other components can mutate the data-objects.
- **Bugs will be introduced.**

Costs
- Figuring out data-structures &rarr; Development time
- QA will find some bugs &rarr; Iteration costs
- Bugs will face the customer.

## The Solution: Enforced Types + Verification

**Remove any &rarr; No more guesswork &rarr; Compiler catches Bugs**

Invest in 
- Defining Data Schemas and Verification
- Typings for data
- Disallow mutation of data-objects

ERTS
- Recommends `io-ts` for Data Schemas and Verification
- Recommends `io-ts` for Typings
- Compiler configured to disallow mutations of data-objects


## Worth it?

The Pay Off is huge. If your code base has no typed API-Responses yet.

Seriously. Adopting this will cut your projects bug tickets in half.

This is a huge generalization. Many projects can cut their bug tickets by 60-80%. 
