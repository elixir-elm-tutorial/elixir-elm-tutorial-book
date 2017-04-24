# Our First Game

Let's get started on creating our first minigame with Elm. We want to start
with something really small and simple, but it should have all the elements
that we're looking for. It should be (very) small, self-contained, interactive,
and fun. And we'll work towards tracking a score and sending that data back
to our back-end platform.

## Creating a Folder for Games

Inside our existing `lib/platform/web/elm` folder, let's create a new folder
called `Games` (the Elm convention is to capitalize file and folder names).

Next, we'll have to add it to the `source-directories` in our
`elm-package.json` file that resides in that `elm` folder:

```json
{
    "version": "1.0.0",
    "summary": "helpful summary of your project, less than 80 characters",
    "repository": "https://github.com/user/project.git",
    "license": "BSD3",
    "source-directories": [
        ".",
        "Games"
    ],
    "exposed-modules": [],
    "dependencies": {
        "elm-lang/core": "5.1.1 <= v < 6.0.0",
        "elm-lang/html": "2.0.0 <= v < 3.0.0"
    },
    "elm-version": "0.18.0 <= v < 0.19.0"
}
```

