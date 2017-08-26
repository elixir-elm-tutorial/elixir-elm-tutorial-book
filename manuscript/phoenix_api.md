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
mix phx.gen.json Products Game games title:string:unique description:string thumbnail:string featured:boolean
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
$ mix phx.gen.json Products Game games title:string description:string thumbnail
:string featured:boolean
* creating lib/platform_web/controllers/game_controller.ex
* creating lib/platform_web/views/game_view.ex
* creating test/platform_web/controllers/game_controller_test.exs
* creating lib/platform_web/views/changeset_view.ex
* creating lib/platform_web/controllers/fallback_controller.ex
* creating lib/platform/products/game.ex
* creating priv/repo/migrations/20170826154100_create_games.exs
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

We'll need to follow the instructions that Phoenix provides for us. Open up the
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

We also want to establish a relationship from our `games` table to our
`players` table. To accomplish this, we'll start with the migration that was
created for us in the `priv/repo/migrations/20170826144626_create_games.exs`
file (bearing in mind that your file will have a different name since the
filenames have a datetime associated with them):

```elixir
defmodule Platform.Repo.Migrations.CreateGames do
  use Ecto.Migration

  def change do
    create table(:games) do
      add :title, :string
      add :description, :string
      add :thumbnail, :string
      add :featured, :boolean, default: false, null: false

      timestamps()
    end

  end
end
```

We can see that we're creating a new table called `games` in our database
with the `create table(:games)` syntax.

We also want to form an association from our `players` table to our new `games`
table. So we're going to create a new table called `gameplays` that will store
a `game_id` as a reference to the `games` table, a `player_id` as a reference
to the `players` table, and a `player_score` that will track a player's score
for the current play through the game.

Let's update the `change` function in our migration to include the following:

```elixir
defmodule Platform.Repo.Migrations.CreateGames do
  use Ecto.Migration

  def change do
    create table(:games) do
      add :title, :string
      add :description, :string
      add :thumbnail, :string
      add :featured, :boolean, default: false, null: false

      timestamps()
    end

    create table(:gameplays) do
      add :game_id, references(:games, on_delete: :nothing), null: false
      add :player_id, references(:players, on_delete: :nothing), null: false
      add :player_score, :integer

      timestamps()
    end
  end
end
```

Before we run our migration, let's take a look at the schema files for players
and games. And we'll manually create a new schema for our gameplays.

## Updating the Schemas

First, open the `lib/platform/accounts/player.ex` file, and let's update our
schema with the following:

```elixir
schema "players" do
  many_to_many :games, Game, join_through: Gameplay

  field :display_name, :string
  field :password, :string, virtual: true
  field :password_digest, :string
  field :score, :integer
  field :username, :string

  timestamps()
end
```

We'll also need to add two `alias` lines above:

```elixir
alias Platform.Products.Game
alias Platform.Products.Gameplay
```

This means that we're establishing a
[`many_to_many`](https://hexdocs.pm/ecto/Ecto.Schema.html#many_to_many/3)
relationship between our `players` and `games`, and that we're joining these
through the `Gameplay` module that we'll be creating shortly.

Next, we'll do something similar for the `lib/platform/products/game.ex` file.
We'll add our two `alias` lines at the top:

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

## Creating a New Schema

Finally, we'll create a schema file for our new table. Create a new file called
`gameplay.ex` inside the `lib/platform/products` folder.

```elixir
defmodule Platform.Products.Gameplay do
  use Ecto.Schema
  import Ecto.Changeset
  alias Platform.Products.Gameplay
  alias Platform.Products.Game
  alias Platform.Accounts.Player

  schema "gameplays" do
    has_one :game, Game
    has_one :player, Player

    field :player_score, :integer, default: 0
  end

  @doc false
  def changeset(%Gameplay{} = gameplay, attrs) do
    gameplay
    |> cast(attrs, [:game, :player, :player_score])
    |> validate_required([:game, :player, :player_score])
  end
end
```

Our new schema follows the same general structure as the other ones we were
just working with, but has a couple of new items. We created our `alias` lines
at the top just as we did previously. For our new `gameplays` schema, we're
creating [`has_one`](https://hexdocs.pm/ecto/Ecto.Schema.html#has_one/3)
relationships so that each gameplay will have just one `player` and one `game`.
We also include a `default` value for our `player_score` field. In our
`changeset/2` function, we also make sure to require all of these fields.

## Running Our Migration

We can now run our migration to update the database with `mix ecto.migrate`.
This creates tables for both `games` and `gameplays`.

```shell
$ mix ecto.migrate
17:17:22.048 [info]  == Running Platform.Repo.Migrations.CreateGames.change/0 forward
17:17:22.048 [info]  create table games
17:17:22.053 [info]  create table gameplays
17:17:22.059 [info]  == Migrated in 0.0s
```

Lastly, we'll run our tests to make sure everything is still working:

```shell
$ mix test
Compiling 17 files (.ex)
Generated platform app
...................................

Finished in 0.2 seconds
35 tests, 0 failures

Randomized with seed 532429
```

We're up to a total of 35 tests! Granted, these were created by the Phoenix
generators; but it gives us some level of confidence that our application is
working when the tests are passing.

## Trying Out our JSON API

Let's start up our Phoenix server with `mix phx.server`.

For our players resource, we were using URLs like `http://0.0.0.0:4000/players`
to access the HTML pages. But now that we added a JSON resource, we'll need to
use `/api` in our URLs. Try to access `http://0.0.0.0:4000/api/games` in the
browser. We shouldn't see an error, but we also don't have any game data to
display yet (note that your browser might display JSON data differently):

![Games API with No Data](images/phoenix_api/games_api_with_no_data.png)

## Adding Data Seeds

We can't interact with our game resources the same way we did with players,
because we're only working with JSON and don't have any HTML pages to view.

Let's update our database for both our players and our games so we have some
sample data to work with. Add the following to the bottom of the
`priv/repo/seeds.ex` file:

```elixir
# Players

Platform.Repo.insert!(%Platform.Accounts.Player{display_name: "José Valim", username: "josevalim", password: "josevalim", score: 1000})
Platform.Repo.insert!(%Platform.Accounts.Player{display_name: "Evan Czaplicki", username: "evancz", password: "evancz", score: 2000})
Platform.Repo.insert!(%Platform.Accounts.Player{display_name: "Joe Armstrong", username: "joearms", password: "joearms", score: 3000})

# Games

Platform.Repo.insert!(%Platform.Products.Game{title: "Adventure Game", description: "Adventure game example.", author_id: 1})
Platform.Repo.insert!(%Platform.Products.Game{title: "Driving Game", description: "Driving game example.", author_id: 2})
Platform.Repo.insert!(%Platform.Products.Game{title: "Platform Game", description: "Platform game example.", author_id: 3})
```

Assuming we don't have any local data that we want to keep, we can use this
file to reseed the database with `mix ecto.reset`. This task will drop the
existing database, create a new one, run migrations, and then seed the
database. This is what the full output should look like:

```shell
$ mix ecto.reset
The database for Platform.Repo has been dropped
The database for Platform.Repo has been created

23:00:56.808 [info]  == Running Platform.Repo.Migrations.CreatePlatform.Accounts.Player.change/0 forward
23:00:56.808 [info]  create table accounts_players
23:00:56.816 [info]  == Migrated in 0.0s
23:00:56.847 [info]  == Running Platform.Repo.Migrations.AddFieldsToPlayerAccounts.change/0 forward
23:00:56.847 [info]  alter table accounts_players
23:00:56.849 [info]  create index accounts_players_username_index
23:00:56.851 [info]  == Migrated in 0.0s
23:00:56.866 [info]  == Running Platform.Repo.Migrations.CreatePlatform.Products.Game.change/0 forward
23:00:56.866 [info]  create table products_games
23:00:56.870 [info]  == Migrated in 0.0s
[debug] QUERY OK db=3.4ms
INSERT INTO "accounts_players" ("display_name","score","username","inserted_at","updated_at") VALUES ($1,$2,$3,$4,$5) RETURNING "id" ["José Valim", 1000, "josevalim", {{2017, 4, 10}, {3, 0, 56, 970688}}, {{2017, 4, 10}, {3, 0, 56, 970696}}]
[debug] QUERY OK db=2.3ms
INSERT INTO "accounts_players" ("display_name","score","username","inserted_at","updated_at") VALUES ($1,$2,$3,$4,$5) RETURNING "id" ["Evan Czaplicki", 2000, "evancz", {{2017, 4, 10}, {3, 0, 56, 987478}}, {{2017, 4, 10}, {3, 0, 56, 987484}}]
[debug] QUERY OK db=2.7ms
INSERT INTO "accounts_players" ("display_name","score","username","inserted_at","updated_at") VALUES ($1,$2,$3,$4,$5) RETURNING "id" ["Joe Armstrong", 3000, "joearms", {{2017, 4, 10}, {3, 0, 56, 990100}}, {{2017, 4, 10}, {3, 0, 56, 990105}}]
[debug] QUERY OK db=2.5ms
INSERT INTO "products_games" ("author_id","description","title","inserted_at","updated_at") VALUES ($1,$2,$3,$4,$5) RETURNING "id" [1, "Adventure game example.", "Adventure Game", {{2017, 4, 10}, {3, 0, 56, 993110}}, {{2017, 4, 10}, {3, 0, 56, 993117}}]
[debug] QUERY OK db=2.0ms
INSERT INTO "products_games" ("author_id","description","title","inserted_at","updated_at") VALUES ($1,$2,$3,$4,$5) RETURNING "id" [2, "Driving game example.", "Driving Game", {{2017, 4, 10}, {3, 0, 56, 995898}}, {{2017, 4, 10}, {3, 0, 56, 995903}}]
[debug] QUERY OK db=2.9ms
INSERT INTO "products_games" ("author_id","description","title","inserted_at","updated_at") VALUES ($1,$2,$3,$4,$5) RETURNING "id" [3, "Platform game example.", "Platform Game", {{2017, 4, 10}, {3, 0, 56, 998162}}, {{2017, 4, 10}, {3, 0, 56, 998167}}]
```

And now that we have some data, we should be able to reload the
`http://0.0.0.0:4000/api/games` URL in our browser and see the results:

![Games API with Data](images/phoenix_api/games_api_with_data.png)

## Summary

This was a relatively quick chapter, but we accomplished exactly what we needed
to do. We'll be able to extend the features for our games later, but this
initial work should be perfect for our initial JSON API.

In the next chapter, we'll get an introduction to the Elm language. And we'll
start working towards using the Phoenix JSON API that we built here to supply
data for our Elm application.
