# Our First Game

Let's start creating our first minigame with Elm. We want to begin with
something really small and simple, but it should also have the characteristics
we want for our games. It should be (very) small, self-contained, interactive,
and fun. We also want to add a simple scoring mechanism so we can work towards
tracking player scores and sending that data to our back-end platform.

## Creating an Initial Game File

Inside the `lib/platform/web/elm` folder, let's create a new file named
`Game.elm`. We can initialize it with the following code so we have
something to display on the page:

```elm
module Game exposing (..)

import Html exposing (..)


main : Html msg
main =
    text "Elm Game"
```

## Creating a New Page for Games

On the Phoenix side, we'll need to create a new page where we can display our
game. We'll start by manually creating a new page and route, and later we'll
work towards a more flexible approach.

Inside the `lib/platform/web/templates/page` folder, let's create a new file
called `game.html.eex`. This will be very similar to what we did for our Elm
home page, and this time we'll just be creating a container div element for our
Elm game. This is all we need to add for the `game.html.eex` file:

```embedded_elixir
<div class="elm-game-container"></div>
```

Now let's update our `router.ex` file inside the `lib/platform/web` folder.
For now, we can add a simple route so that users can access our game by going
to `/elm/game` in the browser. Update the `router.ex` file with the following:

```elixir
scope "/", Platform.Web do
  pipe_through :browser

  get "/", PlayerController, :new
  get "/elm", PageController, :index
  get "/elm/game", PageController, :game
  resources "/players", PlayerController
  resources "/sessions", PlayerSessionController, only: [:new, :create, :delete]
end
```

Now when we visit `http://0.0.0.0:4000/elm/game` in our browser, we'll be able
to see the Elm game that we're creating. But we still need to take a couple
more steps before this works properly.

Let's update our `PageController` so it knows about our new Elm game. In the
`lib/platform/web/controllers` folder, update the `page_controller.ex` file
with the following:

```elixir
defmodule Platform.Web.PageController do
  use Platform.Web, :controller

  plug :authenticate when action in [:index]

  def index(conn, _params) do
    render conn, "index.html"
  end

  def game(conn, _params) do
    render conn, "game.html"
  end

  defp authenticate(conn, _opts) do
    if conn.assigns.current_user() do
      conn
    else
      conn
      |> put_flash(:error, "You must be logged in to access that page.")
      |> redirect(to: player_path(conn, :new))
      |> halt()
    end
  end
end
```

We just need to update our `app.js` file so that it knows to render
our game inside the Phoenix application. At the bottom of our
`assets/js/app.js` file, let's update our code to look like this:

```javascript
const elmContainer = document.querySelector(".elm-container");
const elmGameContainer = document.querySelector(".elm-game-container");

if (elmContainer) {
  const elmApplication = Elm.Main.embed(elmContainer);
}

if (elmGameContainer) {
  const elmGame = Elm.Game.embed(elmGameContainer);
}
```

We also need to update our `brunch-config.js` file:

```javascript
exports.config = {
  files: {
    javascripts: { joinTo: "js/app.js" },
    stylesheets: { joinTo: "css/app.css" },
    templates: { joinTo: "js/app.js" }
  },
  conventions: { assets: /^(static)/ },
  paths: {
    watched: ["../lib/platform/web/elm", "static", "css", "js", "vendor"],
    public: "../priv/static"
  },
  plugins: {
    babel: { ignore: [/vendor/] },
    elmBrunch: { elmFolder: "../lib/platform/web/elm", mainModules: ["Main.elm", "Game.elm"], outputFolder: "../../../../assets/vendor" }
  },
  modules: { autoRequire: { "js/app.js": ["js/app"] } },
  npm: { enabled: true }
};
```

