# Chapter 2: Elm Setup

We're so excited to have our back-end up and running, but before we get too far,
let's start taking a look at setting up our front-end too.

We're going to use Elm for _everything_. Because it's amazing. Elm will offer
guarantees and features that other languages are completely unable to provide
for us. We're going to love it.

## Install Elm

```bash
npm install -g elm
```

## Configuring Elm within Phoenix

```bash
npm install --save-dev elm elm-brunch 
```

## brunch-config.js

```javascript
exports.config = {
  files: {
    javascripts: { joinTo: "js/app.js" },
    stylesheets: { joinTo: "css/app.css", order: { after: ["web/static/css/app.css"] } },
    templates: { joinTo: "js/app.js" }
  },
  conventions: { assets: /^(web\/static\/assets)/ },
  paths: { watched: [ "web/elm", "web/static", "test/static" ], public: "priv/static" },
  plugins: {
    babel: { ignore: [/web\/static\/vendor/] },
    elmBrunch: { elmFolder: "web/elm", mainModules: ["Main.elm"], outputFolder: "../static/vendor" }
  },
  modules: { autoRequire: { "js/app.js": ["web/static/js/app"] } },
  npm: { enabled: true }
};
```


## Creating Our JSON API

```bash
mix phoenix.gen.json Api.Player players --no-model
```

## Updating the Player API Controller

```elixir
# ...
alias Platform.Player
# ...
```

## Updating the Router

```elixir
scope "/api", Platform do
	pipe_through :api
	
	resources "/players", Api.PlayerController, except: [:new, :edit]
end
```

## Updating the Player API View

```elixir
defmodule Platform.Api.PlayerView do
  use Platform.Web, :view

  def render("index.json", %{players: players}) do
    %{data: render_many(players, Platform.Api.PlayerView, "player.json")}
  end

  def render("show.json", %{player: player}) do
    %{data: render_one(player, Platform.Api.PlayerView, "player.json")}
  end

  def render("player.json", %{player: player}) do
    %{id: player.id, username: player.username, score: player.score}
  end
end
```