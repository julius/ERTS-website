# Libraries

Recommended frameworks for frontend projects (included in boilerplate):
- **React** - DOM programming with very mature Typings.
- **io-ts** - Decode JSON-Data safely &rarr; [Best Practices - Typing JSON](best-json.md)

ERTS Buildsystem uses:
- webpack
- typescript
- tslint
...

webpack-dev-server is configured to display any findings of the typescript compiler or tslint in the browser.

## Boilerplate
```bash
git clone git@github.com:julius/ERTS-boilerplate.git && cd ERTS-boilerplate
yarn install
yarn start
```

A premade basic React web application will start up. It already demos typical tasks like 
- Loading data from an API
- Routing
- Translations

[https://github.com/julius/ERTS-boilerplate](https://github.com/julius/ERTS-boilerplate)



## ERTS and React
For DOM-Programming. ERTS can use any library.

DOM-Programming involves a lot of stuff with a lot of types.
React is recommended for safety because of mature typings.

