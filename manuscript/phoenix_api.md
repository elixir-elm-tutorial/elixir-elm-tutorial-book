# Phoenix API

We now have players for our gaming platform, but we don't have any games yet.
We're going to create an Elm application for our front-end that will display
our list of games. But before we get to that, we'll need to create the back-end
JSON API.

## Generating the JSON API

Let's create an endpoint for our games. We want each game to have fields for
`title`, `description`, and `author_id`. We can also add other fields
later, but this will be a good start.

We'll run the Phoenix generator from the Terminal to get started:

```shell
mix phx.gen.json Products Game games title:string description:string author_id:integer
```

This is similar to the way we created our players resource, but this time we're
using `phx.gen.json` instead of `phx.gen.html`.

Also note that we're keeping our contexts intentionally abstract in this book.
Although we're building features that are specific to our gaming domain
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
