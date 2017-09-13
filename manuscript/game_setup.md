# Game Setup

We have our platform application layout in place, and we currently have a
sample "Platformer" game title to work with. But if we click the link to this
game, nothing happens yet because we haven't wired up the game routes and
configuration.

In this chapter, we'll configure our application so we can create new games
inside our `assets/elm` folder, and then load them in the browser through our
platform.

## Creating a Game File

Let's start by creating a new file for our "Platformer" game in the
`assets/elm` folder. We can use the game's title as our Elm module name, so
let's call the file `Platformer.elm` and initialize it with the following code
so we have something to display on the page:

```elm
module Platformer exposing (..)

import Html exposing (..)


main : Html msg
main =
    text "Platformer Game"
```

## Configuring Elm Brunch

Now that we have mutiple Elm source code files, we'll need to update the
`elmBrunch` section of our `brunch-config.js` file. First, we'll add
`"elm/Platformer.elm"` to the list of `mainModules`. Then, we'll add an
`outputFile` property to compile all of our Elm source code into a single
`elm.js` file.

```javascript
elmBrunch: {
  mainModules: ["elm/Main.elm", "elm/Platformer.elm"],
  makeParameters: ["--debug"],
  outputFile: "elm.js",
  outputFolder: "../assets/js"
}
```

When we run our Phoenix server, our application will compile all the Elm
source code and output it to the new `elm.js` file.

We can also include our new output file in the `.gitignore` file at the root of
our project.

```gitignore
# Elm
/assets/elm-stuff
/assets/js/elm.js
```

Feel free to delete the existing `main.js` file that's no longer needed.

## Updating app.js

With our new approach, we can now refactor our `assets/js/app.js` file. We'll
use a single `require()` statement to pull in our compiled `elm.js` output.

```javascript
// Elm
const Elm = require("./elm.js");

const elmContainer = document.querySelector("#elm-container");

if (elmContainer) Elm.Main.embed(elmContainer);
```

With the way we configured our application, we can now use `Elm` as a top-level
namespace, and then embed our code using our module names. In the code above,
we're using `Elm.Main`, and we can also use `Elm.Platformer` to work with our
new game.

```javascript
// Elm
const Elm = require("./elm.js");

const elmContainer = document.querySelector("#elm-container");
const platformer = document.querySelector("#platformer");

if (elmContainer) Elm.Main.embed(elmContainer);
if (platformer) Elm.Platformer.embed(platformer);
```

Keep in mind we haven't created the `div` element with a `#platformer` id yet.
Once we create that element, we'll be able to embed our game using
`Elm.Platformer.embed(platformer)`.

## Extending Our GameController

We've taken care of most of the tedious configuration steps. Now, let's take a
look at our `PlatformWeb.GameController` module in the
`lib/platform_web/controllers` folder. This file currently contains functions
that we use for our JSON API. What we want to do now is to basically add a new
version of the "show" function that allows us to display a page in the browser
for our games.

Below the `show/2` function, let's add a new function called `play/2`. We'll
take the same approach as we do for showing the JSON associated with a game,
but this time we'll use `"show.html"` instead.

```elixir
def show(conn, %{"id" => id}) do
  game = Products.get_game!(id)
  render(conn, "show.json", game: game)
end

def play(conn, %{"id" => id}) do
  game = Products.get_game!(id)
  render(conn, "show.html", game: game)
end
```

## Adding a Route

Now that we have our controller action, we can add a line to our router in the
`lib/platform_web/router.ex` file. We'll route to the `play/2` function in our
`GameController` based on the `:id` for each game:

```elixir
scope "/", PlatformWeb do
  pipe_through :browser

  get "/", PageController, :index
  get "/games/:id", GameController, :play
  resources "/players", PlayerController
  resources "/sessions", PlayerSessionController, only: [:new, :create, :delete]
end
```

## Creating a Template

We also need to add a new template file for our games. Inside the
`lib/platform_web/templates` folder, create a new folder called `game`. Then,
create a file called `show.html.eex` with the following line:

```embedded_elixir
<div id="<%= @game.title |> String.downcase %>"></div>
```

What we're doing here is using the game's `title` field to dynamically create
a new `div` element with an ID that matches that game's title. That allows us
to use the `document.querySelector("#platformer")` line that we used in our
`app.js` code above.

At this point, we should now have everything working well enough to see the
game being rendered in the browser!

![Display Game](images/game_setup/display_game.png)

When we visit `http://0.0.0.0:4000`, we see the Elm application we created in
`Main.elm`. And when we visit `http://0.0.0.0:4000/games/1`, we see the new
game we're going to create in `Platformer.elm`.

## Working with Slugs

Let's clean things up a little bit. Rather than working with our game's `title`
field, we can add a new field called `slug`.

...

We're currently piping our `@game.title` to
`String.downcase` to create an id. But let's create a new function in our
`lib/platform_web/views/game_view.ex` file called `slugify/1` that takes in our
game title and converts it to a "dasherized" slug. Let's add the following
private function to the bottom of our `PlatformWeb.GameView` module:

```elixir
defp slugify(string) do
  string
  |> String.downcase
  |> String.replace(" ", "-")
  |> String.replace(~r/[!.?']/, "")
end
```

In our case, our first game is just called "Platformer", so this isn't needed
yet. But when we create new game titles, we want to be able to work with the
slugs so a title like "Hello World" would become "hello-world".

To use our new view function, let's go back to the
`lib/platform_web/templates/show.html.eex` file and refactor with the following
code:

```embedded_elixir
<div id="<%= slugify(@game.title) %>"></div>
```

## Pretty URLs

products.ex

```elixir
def get_game!(id), do: Repo.get!(Game, id)
def get_game_by_title!(title), do: Repo.get_by!(Game, title: title)
```

game_controller.ex

```elixir
def play(conn, %{"title" => title}) do
  game = Products.get_game_by_title!(title)
  render(conn, "show.html", game: game)
end
```

router.ex

```
scope "/", PlatformWeb do
  pipe_through :browser

  get "/", PageController, :index
  get "/games/:title", GameController, :play
  resources "/players", PlayerController
  resources "/sessions", PlayerSessionController, only: [:new, :create, :delete]
end
```

## Working Links

You may have noticed that we still haven't accomplished our original goal from
the beginning of the chapter.