# Phoenix API and Ecto Relationships

We now have players for our gaming platform, but we don't have any games yet.
We're going to create an Elm application for our front-end that will display
our lists of players and games, but first let's create the back-end JSON API.
And we'll find a way to establish relationships between our players and games
so that we can model our data properly.

## Generating the JSON API

Let's begin by creating an endpoint for our games. We want each game to have
fields for `title`, `description`, `thumbnail`, and `featured`. These fields
will allow us to display a list of games for players to choose from, and the
`featured` field will be a simple boolean value we can use to feature
particular games in a special section. We can also add other fields later, but
this will be a good start.

We'll run the Phoenix generator from the command line to get started:

```shell
$ mix phx.gen.json Products Game games description:string featured:boolean thumbnail:string title:string
```

This is similar to the way we created our players resource, but this time we're
using
[`phx.gen.json`](https://hexdocs.pm/phoenix/Mix.Tasks.Phx.Gen.Json.html)
instead of
[`phx.gen.html`](https://hexdocs.pm/phoenix/Mix.Tasks.Phx.Gen.Html.html).

Also note that we're keeping our contexts intentionally abstract in this book.
Although we're building features that are specific to our gaming domain
(`players` and `games`), we want to be able to adapt this same material for
other uses too. We're using the `Accounts` context for our `players` and the
`Products` context for our `games`, and we could easily use these same concepts
to the domain of a bookstore where our `Accounts` might be `readers` and our
`Products` could be `books`.

Here's what the output should look like when we run the generator for our new
games resource:

```shell
$ mix phx.gen.json Products Game games description:string featured:boolean thumbnail:string title:string
* creating lib/platform_web/controllers/game_controller.ex
* creating lib/platform_web/views/game_view.ex
* creating test/platform_web/controllers/game_controller_test.exs
* creating lib/platform_web/views/changeset_view.ex
* creating lib/platform_web/controllers/fallback_controller.ex
* creating lib/platform/products/game.ex
* creating priv/repo/migrations/20181112124229_create_games.exs
* creating lib/platform/products/products.ex
* injecting lib/platform/products/products.ex
* creating test/platform/products/products_test.exs
* injecting test/platform/products/products_test.exs

Add the resource to your :api scope in lib/platform_web/router.ex:

    resources "/games", GameController, except: [:new, :edit]

Remember to update your repository by running migrations:

    $ mix ecto.migrate
```

Let's hold off on running the Ecto migration, because we'll want to make a few
changes in the next sections first.

## API Routing

Let's follow the instructions that Phoenix provides for us. Open up the
`lib/platform_web/router.ex` file. Instead of adding to the browser scope like
we did previously, we're going to add this resource to the `/api` scope. This
means our two scopes will look like this:

```elixir
scope "/", PlatformWeb do
  pipe_through :browser

  get "/", PageController, :index
  resources "/players", PlayerController
  resources "/sessions", PlayerSessionController, only: [:new, :create, :delete]
end

scope "/api", PlatformWeb do
  pipe_through :api

  resources "/games", GameController, except: [:new, :edit]
end
```

## Establishing Relationships

We generated a new migration for our `games` table above. But, we also want to
form an association from our `players` table to our new `games` table. We're
going to create a new table called `gameplays` that will store a `game_id` as a
reference to the `games` table, a `player_id` as a reference to the `players`
table, and a `player_score` that will track a player's score for the current
play through the game.

Let's take a look at the schema files for players and games. Then, we'll create
a new schema for our gameplays.

First, open the `lib/platform/accounts/player.ex` file, and update our schema
with the following:

```elixir
schema "players" do
  many_to_many :games, Game, join_through: Gameplay

  field :display_name, :string
  field :password, :string, virtual: true
  field :password_digest, :string
  field :score, :integer, default: 0
  field :username, :string, unique: true

  timestamps()
end
```

We'll also need to add two `alias` lines above for the `Game` and `Gameplay`
modules to work:

```elixir
alias Platform.Products.Game
alias Platform.Products.Gameplay
```

This means that we're establishing a
[`many_to_many`](https://hexdocs.pm/ecto/Ecto.Schema.html#many_to_many/3)
relationship between our `players` and `games`, and that we're joining these
through the `Gameplay` module that we'll be creating shortly.

Next, we'll do something similar for the `lib/platform/products/game.ex` file.
We'll add our two `alias` lines for `Gameplay` and `Player` at the top:

```elixir
alias Platform.Products.Gameplay
alias Platform.Accounts.Player
```

Then, we'll add our `many_to_many` relationship between our `games` and our
`players` through the `Gameplay` module.

```elixir
schema "games" do
  many_to_many :players, Player, join_through: Gameplay

  field :description, :string
  field :featured, :boolean, default: false
  field :thumbnail, :string
  field :title, :string

  timestamps()
end
```

## Creating Gameplays

Now, we can create the `Gameplay` part of our application that connects our
players and games. We're using the same `mix phx.gen.json` generator we used
for our games, but this time we're using `references` to associate our
`gameplay` records with `player_id` and `game_id` fields.

```shell
$ mix phx.gen.json Products Gameplay gameplays game_id:references:games player_id:references:players player_score:integer
```

Keep in mind that we could manually create many of these files and custom build
these features into our application. But using these generators gives us a lot
of scaffolding that allows us to move quickly, and it will save us a _ton_ of
effort later in the book when we're ready to display our player scores.

Here is what the output should look like for our generator command. We'll be
prompted with a warning about overwriting some files, but we can just enter the
`Y` character to continue since we just generated those files for games.

```shell
$ mix phx.gen.json Products Gameplay gameplays game_id:references:games player_id:references:players player_score:integer
The following files conflict with new files to be generated:

  * lib/platform_web/views/changeset_view.ex
  * lib/platform_web/controllers/fallback_controller.ex

See the --web option to namespace similarly named resources

Proceed with interactive overwrite? [Yn] Y
* creating lib/platform_web/controllers/gameplay_controller.ex
* creating lib/platform_web/views/gameplay_view.ex
* creating test/platform_web/controllers/gameplay_controller_test.exs
* creating lib/platform/products/gameplay.ex
* creating priv/repo/migrations/20181112124950_create_gameplays.exs
* injecting lib/platform/products/products.ex
* injecting test/platform/products/products_test.exs

Add the resource to your :api scope in lib/platform_web/router.ex:

    resources "/gameplays", GameplayController, except: [:new, :edit]


Remember to update your repository by running migrations:
    $ mix ecto.migrate
```

Next, we'll go ahead and add the `resources` line that we see in the output
above to our `lib/platform_web/router.ex` file. Here is the `"/api"` scope for
our router:

```elixir
scope "/api", PlatformWeb do
  pipe_through :api

  resources "/games", GameController, except: [:new, :edit]
  resources "/gameplays", GameplayController, except: [:new, :edit]
end
```

## Our New Gameplay Schema

Let's open the `lib/platform/products/gameplay.ex` file to take a look at our
new schema.

```elixir
defmodule Platform.Products.Gameplay do
  use Ecto.Schema
  import Ecto.Changeset

  alias Platform.Products.Gameplay

  schema "gameplays" do
    field :player_score, :integer
    field :game_id, :id
    field :player_id, :id

    timestamps()
  end

  @doc false
  def changeset(gameplay, attrs) do
    gameplay
    |> cast(attrs, [:player_score])
    |> validate_required([:player_score])
  end
end
```

We used `many_to_many` relationships in both our `Player` schema and our `Game`
schema. For our `Gameplay` schema, we'll use
[`belongs_to`](https://hexdocs.pm/ecto/Ecto.Schema.html#belongs_to/3) to
form the association. We'll also add a few `alias` statements and add a
`default: 0` value for the `player_score` field so we don't end up storing
`nil` values in the database.

```elixir
defmodule Platform.Products.Gameplay do
  use Ecto.Schema
  import Ecto.Changeset

  alias Platform.Products.Game
  alias Platform.Accounts.Player

  schema "gameplays" do
    belongs_to :game, Game
    belongs_to :player, Player

    field :player_score, :integer, default: 0

    timestamps()
  end

  @doc false
  def changeset(gameplay, attrs) do
    gameplay
    |> cast(attrs, [:player_score])
    |> validate_required([:player_score])
  end
end
```

## Running Our Migration

We can now run our migration to update the database with `mix ecto.migrate`.
This creates tables for both `games` and `gameplays`.

```shell
$ mix ecto.migrate
Compiling 21 files (.ex)
Generated platform app
[info] == Running Platform.Repo.Migrations.CreateGames.change/0 forward
[info] create table games
[info] == Migrated in 0.0s
[info] == Running Platform.Repo.Migrations.CreateGameplays.change/0 forward
[info] create table gameplays
[info] create index gameplays_game_id_index
[info] create index gameplays_player_id_index
[info] == Migrated in 0.0s
```

Lastly, we'll run our tests to make sure everything is still working:

```shell
$ mix test
................................................

Finished in 0.5 seconds
47 tests, 0 failures
```

It's great that we have 47 passing tests. Granted, these were created by the
Phoenix generators, but it gives us some level of confidence that our
application is working when the tests are passing.

## Trying Out our JSON API

Let's start up our Phoenix server with `mix phx.server`.

For our players resource, we were using URLs like `http://localhost:4000/players`
to access the templates. But now that we added a JSON resource, we'll need to
use `/api` in our URLs. Try to access `http://localhost:4000/api/games` in the
browser. We shouldn't see an error, but we also don't have any game data to
display yet (note that your browser might display JSON data differently):

![Games API with No Data](images/phoenix_api/games_api_with_no_data.png)

## Creating a Game

We can't interact with our game resources the same way we did with players,
because we're only working with JSON and don't have any HTML pages to view.

Instead, let's fire up an `iex` session and create a new game from the command
line. First, enter the following at the command line to get started:

```shell
$ iex -S mix phx.server
```

From the Terminal, we can create a new game called "Platformer" manually by
using the `create_game/1` function from the `Platform.Products` module. If you
can't see the `iex>` prompt, you can press the ENTER key in the Terminal to
see a new prompt.

```elixir
iex> Platform.Products.create_game(%{title: "Platformer", description: "Platform game example.", thumbnail: "http://via.placeholder.com/300x200", featured: true})
```

We're setting values for the game's `title`, `description`, `thumbnail`, and
`featured` fields. And the output should look something like this:

```elixir
iex> Platform.Products.create_game(%{title: "Platformer", description: "Platform game example.", thumbnail: "http://via.placeholder.com/300x200", featured: true})
# ...
{:ok,
 %Platform.Products.Game{__meta__: #Ecto.Schema.Metadata<:loaded, "games">,
  description: "Platform game example.", featured: true, id: 1,
  inserted_at: ~N[2018-12-04 15:16:16.957673],
  players: #Ecto.Association.NotLoaded<association :players is not loaded>,
  thumbnail: "http://via.placeholder.com/300x200",
  title: "Platformer", updated_at: ~N[2018-12-04 15:16:16.967729]}}
```

Now that we have some data, we should be able to reload the
`http://localhost:4000/api/games` URL in our browser to see the results:

![Games API with Data](images/phoenix_api/games_api_with_data.png)

## Player API

We had previously been using the browser to work with our player accounts. But
since we're transitioning to an API for our games, we'll also want to make our
players resource accessible as JSON too.

Let's create a new controller in the `lib/platform_web/controllers` folder.
We'll call the file `player_api_controller.ex` and then add the following
contents:

```elixir
defmodule PlatformWeb.PlayerApiController do
  use PlatformWeb, :controller

  alias Platform.Accounts
  alias Platform.Accounts.Player

  action_fallback PlatformWeb.FallbackController

  def index(conn, _params) do
    players = Accounts.list_players()
    render(conn, "index.json", players: players)
  end

  def create(conn, %{"player" => player_params}) do
    with {:ok, player} <- Accounts.create_player(player_params) do
      conn
      |> put_status(:created)
      |> put_resp_header("location", Routes.player_path(conn, :show, player))
      |> render("show.json", player: player)
    end
  end

  def show(conn, %{"id" => id}) do
    player = Accounts.get_player!(id)
    render(conn, "show.json", player: player)
  end

  def update(conn, %{"id" => id, "player" => player_params}) do
    player = Accounts.get_player!(id)

    with {:ok, player} <- Accounts.update_player(player, player_params) do
      render(conn, "show.json", player: player)
    end
  end

  def delete(conn, %{"id" => id}) do
    player = Accounts.get_player!(id)

    with {:ok, %Player{}} <- Accounts.delete_player(player) do
      send_resp(conn, :no_content, "")
    end
  end
end
```

This may look like a lot of code, but we're essentially copying the same thing
that the Phoenix generators gave us for our games API and making small
adjustments so that we can work with accounts and players instead of products
and games.

We'll also want to update our `lib/platform_web/router.ex` file with the new
resource:

```elixir
scope "/api", PlatformWeb do
  pipe_through :api

  resources "/games", GameController, except: [:new, :edit]
  resources "/gameplays", GameplayController, except: [:new, :edit]
  resources "/players", PlayerApiController, except: [:new, :edit]
end
```

To finish making our players accessible via a JSON API, we need to add a view.
Create a file named `player_api_view.ex` inside the `lib/platform_web/views`
folder and add the following content:

```elixir
defmodule PlatformWeb.PlayerApiView do
  use PlatformWeb, :view

  alias PlatformWeb.PlayerApiView

  def render("index.json", %{players: players}) do
    %{data: render_many(players, PlayerApiView, "player.json")}
  end

  def render("show.json", %{player: player}) do
    %{data: render_one(player, PlayerApiView, "player.json")}
  end

  def render("player.json", %{player_api: player_api}) do
    %{
      id: player_api.id,
      username: player_api.username,
      display_name: player_api.display_name,
      score: player_api.score
    }
  end
end
```

Our `PlayerApiView` module is similar to what we have in the `GameView` module.
When we load the `http://localhost:4000/api/players` URL, we're using
`render_many/3` to list all the players. When we only want to show a single
player, we can use a URL like `http://localhost:4000/api/players/1` that will use
`render_one/3` to only display a single user's JSON data. At the bottom, we're
creating a function that returns a map with all our player data. We can add or
remove fields here whenever we want to adjust the fields that are accessible
via the JSON API.

This is all great news because it means we can still use the
`http://localhost:4000/players` URL to access our list of players in the browser,
and we can use `http://localhost:4000/api/players` to see our player data as
JSON.

![Player Data in Browser Scope](images/phoenix_api/player_data_browser_scope.png)

![Player Data in API Scope](images/phoenix_api/player_data_api_scope.png)

Now would be a good time to commit changes to your Git repository since we've
come a long way in this chapter.

## Summary

We managed to accomplish our goal of creating a JSON API for the games on our
platform. And we also learned about Ecto relationships as we connected our
players, games, and gameplays together.

In the next chapter, we'll get an introduction to the Elm language. And we'll
start working towards using the Phoenix JSON API that we built here to supply
data for our Elm application.
