# Component Structure

## Long files vs Complex code

**Long ERTS files &rarr; Simple code**

Complex code
- many complicated if-statements
- many nested if-branches
- no data-structure documentation
- takes considerable time to understand

Simple code
- few if-branches (no nesting aka spaghetti code)
- mostly lists
- data-structures well documented (eg types)
- quickly find the code-section of interest

Using ERTS code files are often long, because of
- Union Types
    - Replaces multiple booleans &rarr; No more nested/complicated ifs
    - Flat list branching
    - Simple code

- Type definitions
    - Less Bugs
    - Data-Structure documentation (eg IntelliSense)
    - Simple Code

**Long files are not an anti-pattern with ERTS.**

## Refactoring
**Never separate code prematurely.** 

Refactoring is cheap
- IDEs will provide typical TypeScript refactorings
- ERTS code is extremely typed &rarr; high confidence refactorings
- If it compiles &rarr; the refactoring was successful

Do not separate code
- at planning stage
- when files get too long *(not an anti-pattern with ERTS!)*


**Good reasons to refactor**

Split up a component, when the state can no longer be expressed well with one Union Type.

Any refactoring of components should always focus on the state. Splitting up the state. Do not focus on the number of functions.

## Typed React.Component

Always use `ERTSComponent` instead of `React.Component`. 
It is exactly the same thing. The only difference is with the typing of `setState`

`React.Component -> setState(state)`
- state can almost be anything
- partial states and invalid states are allowed
- components are NOT prevented from entering invalid states

`ERTSComponent -> setState(state)`
- state is properly type-checked
- state can not be partial &rarr; important for union-type states
- components are prevented from entering invalid states

*No Lock-In: ERTSComponent can later be replaced with React.Component. Simple find&replace. Nothing will break.*
