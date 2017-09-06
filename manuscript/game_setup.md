# Game Setup

...

## Current Approach

- Add game file to `assets/elm` folder.

```
assets/elm/Platformer.elm
```

- Add module to `assets/brunch-config.js`.

```
mainModules: ["elm/Main.elm", "elm/Platformer.elm"],
```

- Add import, query, and embed to `assets/app.js`.

```javascript
// Elm
const Elm = require("./elm.js");

const elmContainer = document.querySelector("#elm-container");
const platformer = document.querySelector("#Platformer");
const adventure = document.querySelector("#Adventure");

if (elmContainer) Elm.Main.embed(elmContainer);
if (platformer) Elm.Platformer.embed(platformer);
if (adventure) Elm.Adventure.embed(adventure);
```