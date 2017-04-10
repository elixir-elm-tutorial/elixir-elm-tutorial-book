# Phoenix API

We now have players for our gaming platform, but we don't have any games yet.
We're going to create an Elm single page application to handle our front-end
code. But before we get to that, we'll need to create the back-end JSON API.

## Generating the JSON API

Let's create an endpoint for our games. We want them to have fields for the
game's `title`, `description`, and `author_id`. We can also add other fields
later, but this will be a good start.

We'll run the Phoenix generator from the Terminal to get started:

```shell
mix phx.gen.json Products Game games title:string description:string author_id:integer
```

This is similar to the way we created our players resource, but this time we're
using `phx.gen.json` instead of `phx.gen.html`.

Also note that we're keeping our contexts intentionally abstract in this book.
Although we're building features that are very specific to our gaming domain
(`players` and `games`), we want to be able to adapt this same material for
other uses too. We're using the `Accounts` context for our `players` and the
`Products` context for our `games`, and we could easily use these same concepts
to the domain of a bookstore where our `Accounts` might be `readers` and our
`Products` would be `books`.

Here's what the output should look like when we run the generator for our new
games resource:

```shell
$ mix phx.gen.json Products Game games title:string description:string author_id:integer
* creating lib/platform/web/controllers/game_controller.ex
* creating lib/platform/web/views/game_view.ex
* creating test/web/controllers/game_controller_test.exs
* creating lib/platform/web/views/changeset_view.ex
* creating lib/platform/web/controllers/fallback_controller.ex
* creating lib/platform/products/game.ex
* creating priv/repo/migrations/20170410023755_create_products_game.exs
* creating lib/platform/products/products.ex
* creating test/products_test.exs

Add the resource to your api scope in lib/platform/web/router.ex:

    resources "/games", GameController, except: [:new, :edit]

Remember to update your repository by running migrations:

    $ mix ecto.migrate
```

We'll need to follow the instructions that Phoenix provides for us. Open up the
`lib/platform/web/router.ex` file. Instead of adding to the browser scope like
we did previously, we're going to add this resource to the `/api` scope. This
means our two scopes will look like this:

```elixir
scope "/", Platform.Web do
  pipe_through :browser

  get "/", PageController, :index
  resources "/players", PlayerController
  resources "/sessions", PlayerSessionController, only: [:new, :create, :delete]
end

scope "/api", Platform.Web do
  pipe_through :api

  resources "/games", GameController, except: [:new, :edit]
end
```

Now that we have our resources generated and they're added to our router, we
can run our migration to update the database with `mix ecto.migrate`. This
creates the `products_games` table in the database:

```shell
$ mix ecto.migrate
Compiling 16 files (.ex)
Generated platform app

22:45:17.194 [info]  == Running Platform.Repo.Migrations.CreatePlatform.Products.Game.change/0 forward
22:45:17.194 [info]  create table products_games
22:45:17.208 [info]  == Migrated in 0.0s
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

Let's start up our Phoenix server (or restart if it was already running) with
`mix phx.server`.

For our player resources, we were using URLs like `http://0.0.0.0:4000/players`
to access the HTML pages. But now that we added a JSON resource, we'll need to
use `/api`. Try to access `http://0.0.0.0:4000/api/games` in the browser. We
shouldn't see an error, but we also don't have any game data to display yet
(note that your browser might display JSON data slightly different):

![Games API with No Data](images/phoenix_api/games_api_with_no_data.png)

## Adding Data Seeds

We can't interact with our game resources the same way we did with players,
because we're only working with JSON and don't have any HTML pages to view. But
let's create some sample data to work with in the `priv/repo/seeds.ex` file:

```elixir
# Players

Platform.Repo.insert!(%Platform.Players.Player{display_name: "José Valim", username: "josevalim", password: "password", score: 1000})
Platform.Repo.insert!(%Platform.Players.Player{display_name: "Evan Czaplicki", username: "evancz", password: "password", score: 1500})

# Games

Platform.Repo.insert!(%Platform.Games.Game{title: "Platform Game", description: "Platform game example.", author_id: 1})
Platform.Repo.insert!(%Platform.Games.Game{title: "Adventure Game", description: "Adventure game example.", author_id: 2})
```

Assuming we don't have any local data that we want to keep, we can use this to
reseed the database with `mix ecto.reset`. This task will drop the existing
database, create a new one, run migrations, and then seed the database.

```shell
$ mix ecto.reset
The database for Platform.Repo has been dropped
The database for Platform.Repo has been created

11:41:44.167 [info]  == Running Platform.Repo.Migrations.CreatePlatform.Players.Player.change/0 forward
11:41:44.168 [info]  create table players_players
11:41:44.182 [info]  == Migrated in 0.0s
11:41:44.255 [info]  == Running Platform.Repo.Migrations.AddFieldsToPlayers.change/0 forward
11:41:44.255 [info]  alter table players_players
11:41:44.262 [info]  create index players_players_username_index
11:41:44.266 [info]  == Migrated in 0.0s
11:41:44.292 [info]  == Running Platform.Repo.Migrations.CreatePlatform.Games.Game.change/0 forward
11:41:44.292 [info]  create table games_games
11:41:44.305 [info]  == Migrated in 0.0s

[debug] QUERY OK db=4.0ms
INSERT INTO "players_players" ("display_name","score","username","inserted_at","updated_at") VALUES ($1,$2,$3,$4,$5) RETURNING "id" ["José Valim", 1000, "josevalim", {{2017, 3, 12}, {15, 41, 44, 535203}}, {{2017, 3, 12}, {15, 41, 44, 535213}}]
[debug] QUERY OK db=2.1ms queue=0.1ms
INSERT INTO "players_players" ("display_name","score","username","inserted_at","updated_at") VALUES ($1,$2,$3,$4,$5) RETURNING "id" ["Evan Czaplicki", 1500, "evancz", {{2017, 3, 12}, {15, 41, 44, 562350}}, {{2017, 3, 12}, {15, 41, 44, 562356}}]
[debug] QUERY OK db=2.1ms
INSERT INTO "games_games" ("author_id","description","title","inserted_at","updated_at") VALUES ($1,$2,$3,$4,$5) RETURNING "id" [1, "Platform game example.", "Platform Game", {{2017, 3, 12}, {15, 41, 44, 564871}}, {{2017, 3, 12}, {15, 41, 44, 564879}}]
[debug] QUERY OK db=2.2ms
INSERT INTO "games_games" ("author_id","description","title","inserted_at","updated_at") VALUES ($1,$2,$3,$4,$5) RETURNING "id" [2, "Adventure game example.", "Adventure Game", {{2017, 3, 12}, {15, 41, 44, 567390}}, {{2017, 3, 12}, {15, 41, 44, 567396}}]
```

And now that we have some data we should be able to reload the
`http://0.0.0.0:4000/api/games` URL in our browser and see the results:

![Games API with Data](images/phoenix_api/games_api_with_data.png)

## Summary

We'll need to extend the features for our games later, but for now this will
work as an initial API. And it will be really helpful as we start to build our
Elm application that consumes data from this Phoenix API.
