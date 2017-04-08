# Phoenix Sign Up

There are many different ways to handle authentication in our applications. In
our case, we're eager to start using Elm to build out the front-end for our
application. So a quick way to handle authentication will be to use Phoenix to
extend the Players resource. We'll use Phoenix to allow users to sign up and
sign in. And once they're authenticated, they'll move "into" the main Elm
application we're going to build.

## Extending Players

Let's take a look at the players we created previously. First, check out the
`lib/players/players.ex` file:

```elixir
defmodule Platform.Players do
  @moduledoc """
  The boundary for the Players system.
  """

  import Ecto.{Query, Changeset}, warn: false
  alias Platform.Repo

  alias Platform.Players.Player

  @doc """
  Returns the list of players.

  ## Examples

      iex> list_players()
      [%Player{}, ...]

  """
  def list_players do
    Repo.all(Player)
  end

  ...
```

Inside the `Platform.Players` module, it looks like there's a handy
`list_players/0` function available for us that will grab all the `Player`
records from the database (`Repo`).

We can use this function along with `iex` to interactively query for our
existing player records:

```shell
$ iex -S mix phx.server
```

```elixir
iex> Platform.Players.list_players
[debug] QUERY OK source="players_players" db=13.5ms decode=0.1ms queue=0.2ms
SELECT p0."id", p0."username", p0."score", p0."inserted_at", p0."updated_at" FROM "players_players" AS p0 []
[%Platform.Players.Player{__meta__: #Ecto.Schema.Metadata<:loaded, "players_players">,
  id: 1, inserted_at: ~N[2017-03-04 15:49:04.509015], score: 1000,
  updated_at: ~N[2017-03-04 15:49:04.526918], username: "josevalim"},
 %Platform.Players.Player{__meta__: #Ecto.Schema.Metadata<:loaded, "players_players">,
  id: 2, inserted_at: ~N[2017-03-04 15:49:40.504744], score: 1500,
  updated_at: ~N[2017-03-04 15:49:40.504775], username: "evancz"}]
```

We can also look up a single player using the `get/2` function and the player's
`id` value:

```elixir
iex> Platform.Repo.get(Platform.Players.Player, 1)
[debug] QUERY OK source="players_players" db=8.9ms queue=0.1ms
SELECT p0."id", p0."username", p0."score", p0."inserted_at", p0."updated_at" FROM "players_players" AS p0 WHERE (p0."id" = $1) [1]
%Platform.Players.Player{__meta__: #Ecto.Schema.Metadata<:loaded, "players_players">,
 id: 1, inserted_at: ~N[2017-03-04 15:49:04.509015], score: 1000,
 updated_at: ~N[2017-03-04 15:49:04.526918], username: "josevalim"}
```

## Player Fields

When we created our Players resource, we ran the generator to create two fields:

- `username`
- `score`

It looks like Phoenix also included a couple of other fields by default:

- `id`
- `inserted_at`
- `updated_at`

## Adding Fields

We have some existing fields for our players, but what if we want to add new
fields? We'll not only have to update our application, but also update our
database.

For our players to be able to sign in, we'll need to add a couple more fields:

- `display_name`
- `password`
- `password_hash`

Let's update the `lib/platform/players/player.ex` file to look like this:

```elixir
defmodule Platform.Players.Player do
  use Ecto.Schema
  
  schema "players_players" do
    field :display_name, :string
    field :username, :string
    field :score, :integer
    field :password, :string, virtual: true
    field :password_hash, :string

    timestamps()
  end
end
```

## Generating a Migration

Now we'll need to update the database to know about the new fields we want to
use for our players. Let's run the following command to generate a migration
file:

```shell
$ mix ecto.gen.migration add_fields_to_players
```

And we should see the output that it successfully generated the migration file,
which we'll fill out next:

```shell
$ mix ecto.gen.migration add_fields_to_players
Compiling 3 files (.ex)
* creating priv/repo/migrations
* creating priv/repo/migrations/20170305140256_add_fields_to_players.exs
```

Let's fill out the migration file that we created. We'll `alter` the existing
database table for our players, and add the fields we need. And we'll also add
a `unique_index` at the bottom to ensure we have unique users:

```elixir
defmodule Platform.Repo.Migrations.AddFieldsToPlayers do
  use Ecto.Migration

  def change do
    alter table(:players_players) do
      add :display_name, :string
      add :password, :string
      add :password_hash, :string
    end

    create unique_index(:players_players, [:username])
  end
end
```

## Running the Migration

Now we can run the migration to update our database. Run the following command
and you should see similar output:

```shell
$ mix ecto.migrate

09:10:21.240 [info]  == Running Platform.Repo.Migrations.AddFieldsToPlayers.change/0 forward
09:10:21.240 [info]  alter table players_players
09:10:21.268 [info]  create index players_players_username_index
09:10:21.278 [info]  == Migrated in 0.0s
```

## Updating Our Application

So far so good. We've managed to alter the database so it knows how to deal with
our new data. But we'll also have to update our application to handle these new
fields.

Let's start with what we want our users to do when they first sign up. Open up
the `lib/platform/web/templates/player/new.html.eex` file:

```embedded_elixir
<h2>New Player</h2>

<%= render "form.html", changeset: @changeset,
                        action: player_path(@conn, :create) %>

<span><%= link "Back", to: player_path(@conn, :index) %></span>
```

Hmm. It looks like this is pulling in "form.html", which is shared between our
new user form and our edit user form. We actually want slightly different
behavior for our application. We want users to be able to sign up with minimal
effort by entering only a `username` and `password`. And then once they're
signed up, they can enter additional information like their `display_name` or
perhaps later enter an email or gravatar.

## Working with Forms

Here's what our original New Player form looks like:

![Original New Player Page](images/phoenix_sign_up/phoenix_original_sign_up.png)

We don't actually want our players to be able to manually enter their scores.
We just want players to create a `username` and `password` to sign up (we're
also going to work on authentication in the next sections, so we won't be
storing passwords as plain text).

We're basically going to move part of the "form.html" file over to the
`new.html.eex` file. We'll also create a form in the `edit.html.eex` file, and
ultimately we'll be able to delete the shared `form.html.eex` file as a result.

Let's start by updating our `new.html.eex` file:

```embedded_elixir
<h2>Quick Player Sign Up</h2>

<%= form_for @changeset, player_path(@conn, :create), fn f -> %>
  <%= if @changeset.action do %>
    <div class="alert alert-danger">
      <p>Oops, something went wrong! Please check the errors below.</p>
    </div>
  <% end %>

  <div class="form-group">
    <%= label f, :username, "Player Username", class: "control-label" %>
    <%= text_input f, :username, placeholder: "Enter username...", class: "form-control" %>
    <%= error_tag f, :username %>
  </div>

  <div class="form-group">
    <%= label f, :password, "Password", class: "control-label" %>
    <%= password_input f, :password, placeholder: "Enter password...", class: "form-control" %>
    <%= error_tag f, :password %>
  </div>

  <div class="form-group">
    <%= submit "Submit", class: "btn btn-primary" %>
  </div>
<% end %>

<span><%= link "Back", to: player_path(@conn, :index) %></span>
```

The form looks great, but try to create a new player and we'll run into an
issue.

![Updated New Player Page](images/phoenix_sign_up/updated_new_player_page.png)

## Validations

When we generated our players resource, it automatically created some default
validations for the fields we specified. That means our Phoenix application
considers `username` and `score` to be required fields to create an account.
Let's take a look at the bottom of the `lib/platform/players/players.ex` file:

```elixir
defmodule Platform.Players do
  # ...

  @doc """
  Returns an `%Ecto.Changeset{}` for tracking player changes.

  ## Examples

      iex> change_player(player)
      %Ecto.Changeset{source: %Player{}}

  """
  def change_player(%Player{} = player) do
    player_changeset(player, %{})
  end

  defp player_changeset(%Player{} = player, attrs) do
    player
    |> cast(attrs, [:username, :score])
    |> validate_required([:username])
  end
end
```

This is our first look at a Phoenix "changeset". When we change a record (or
create a new one like we're doing now), it'll run through some validations.
We'll deal with our passwords in the next few sections, but for now let's remove
the requirement for the `score` field. Now that we removed that field from the
sign up form, users can't fill it in anyway (and we should probably set it to
a default value of `0` when users sign up).

Update the `player_changeset/2` function with the following:

```elixir
defp player_changeset(%Player{} = player, attrs) do
  player
  |> cast(attrs, [:username, :score])
  |> validate_required([:username])
end
```

We now have the ability to create new players! Lastly, let's update the test
that ensures our New Player page is working since we updated the title. We'll
have to update these two test cases to check for "Quick Player Sign Up" instead
of "New Player":

```elixir
test "renders form for new players", %{conn: conn} do
  conn = get conn, player_path(conn, :new)
  assert html_response(conn, 200) =~ "Quick Player Sign Up"
end

test "does not create player and renders errors when data is invalid", %{conn: conn} do
  conn = post conn, player_path(conn, :create), player: @invalid_attrs
  assert html_response(conn, 200) =~ "Quick Player Sign Up"
end
```

## Show Page

After a new user is created, the application currently redirects them to the
player "show" page. When we have the full platform application built out later,
we'll probably want to redirect them to a list of games to play. But for now
we'll update the show page to display all the relevant fields for the player's
account.

Which fields do we want to display on this page?

- `id`
- `display_name`
- `username`
- `score`

We can work towards making this page look nicer with styles and better logic
later, but for now we just want to make sure we can display all the fields we
want. Update the `lib/platform/web/templates/show.html.eex` file with the
following:

```embedded_elixir
<h2>Show Player</h2>

<ul>
  <li><strong>ID: </strong><%= @player.id %></li>
  <li><strong>Display Name: </strong><%= @player.display_name %></li>
  <li><strong>Username: </strong><%= @player.username %></li>
  <li><strong>Score: </strong><%= @player.score %></li>
</ul>

<span><%= link "Edit", to: player_path(@conn, :edit, @player) %></span>
<span><%= link "Back", to: player_path(@conn, :index) %></span>
```

## Edit Page

For the edit page, we're going to do much of the same that we did for the new
player page. We want users to be able to adjust their `username` and
`display_name` fields for example, but they shouldn't be able to manually alter
their `score` field since that should be coming from the games they play on the
platform.

Update the `lib/platform/web/templates/edit.html.eex` file to contain the
following:

```embedded_elixir
<h2>Edit Player</h2>

<%= form_for @changeset, player_path(@conn, :update, @player), fn f -> %>
  <%= if @changeset.action do %>
    <div class="alert alert-danger">
      <p>Oops, something went wrong! Please check the errors below.</p>
    </div>
  <% end %>

  <div class="form-group">
    <%= label f, :display_name, "Player Display Name", class: "control-label" %>
    <%= text_input f, :display_name, placeholder: "Enter display name...", class: "form-control" %>
    <%= error_tag f, :display_name %>
  </div>

  <div class="form-group">
    <%= label f, :username, "Player Username", class: "control-label" %>
    <%= text_input f, :username, placeholder: "Enter username...", class: "form-control" %>
    <%= error_tag f, :username %>
  </div>

  <div class="form-group">
    <%= label f, :password, "Password", class: "control-label" %>
    <%= password_input f, :password, placeholder: "Enter password...", class: "form-control" %>
    <%= error_tag f, :password %>
  </div>

  <div class="form-group">
    <%= submit "Submit", class: "btn btn-primary" %>
  </div>
<% end %>
```

## Shared Form

Now that we've adjusted our New Player form and our Edit Player form, we can go
ahead and delete the `form.html.eex` file that was shared between them.

## Database Seeds

Now that we have all the fields we want to work with, it can be helpful to add
some default data seeds with our application. Instead of manually creating new
database records while we're working in the development environment, this gives
us a quick way to seed the database.

The other benefit of this approach is that when other developers clone our
repository, they'll be able to run the `mix ecto.setup` command and it will
create the database, run migrations, and seed the application with some sample
data.

Open the `priv/repo/seeds.exs` file:

```elixir
# Script for populating the database. You can run it as:
#
#     mix run priv/repo/seeds.exs
#
# Inside the script, you can read and write to any of your
# repositories directly:
#
#     Platform.Repo.insert!(%Platform.SomeSchema{})
#
# We recommend using the bang functions (`insert!`, `update!`
# and so on) as they will fail if something goes wrong.
```

We can see that this file only contains comments initially. But thankfully these
comments are a helpful indication of what we can do to add seeds for our data.
We just want to add a couple of basic seeds for our application:

```elixir
# Script for populating the database. You can run it as:
#
#     mix run priv/repo/seeds.exs
#
# Inside the script, you can read and write to any of your
# repositories directly:
#
#     Platform.Repo.insert!(%Platform.SomeSchema{})
#
# We recommend using the bang functions (`insert!`, `update!`
# and so on) as they will fail if something goes wrong.
Platform.Repo.insert!(%Platform.Players.Player{display_name: "José Valim", username: "josevalim", score: 1000})
Platform.Repo.insert!(%Platform.Players.Player{display_name: "Evan Czaplicki", username: "evancz", score: 1500})
```

And we can see at the top that we can run the following `mix` command to
populate the database with these seeds:

```shell
mix run priv/repo/seeds.exs
```

Keep in mind that we may have already created these records, and we did add a
constraint to our database to make sure that usernames are unique. But the good
news is that we can always adjust this file with some sample data to work with.
And by doing that, we don't have to worry about manually creating records in our
development environment just to test things out.

In fact, there's a command we can use often if we end up creating extraneous
data in our development environment and want to start fresh. If you have records
that you created locally and you don't want to lose them, then DON'T run this
command! But if you ever run into database issues, running `mix ecto.reset` can
be a lifesaver. It drops the existing database (don't say I didn't warn you),
creates a new one, runs all the migrations, and then seeds the database.

Here's what it looks like in action:

```shell
$ mix ecto.reset
The database for Platform.Repo has been dropped
The database for Platform.Repo has been created

10:31:54.538 [info]  == Running Platform.Repo.Migrations.CreatePlatform.Players.Player.change/0 forward
10:31:54.539 [info]  create table players_players
10:31:54.575 [info]  == Migrated in 0.0s
10:31:54.654 [info]  == Running Platform.Repo.Migrations.AddFieldsToPlayers.change/0 forward
10:31:54.654 [info]  alter table players_players
10:31:54.659 [info]  create index players_players_username_index
10:31:54.662 [info]  == Migrated in 0.0s
[debug] QUERY OK db=2.1ms
INSERT INTO "players_players" ("display_name","score","username","inserted_at","updated_at") VALUES ($1,$2,$3,$4,$5) RETURNING "id" ["José Valim", 1000, "josevalim", {
{2017, 3, 5}, {15, 31, 54, 889083}}, {{2017, 3, 5}, {15, 31, 54, 889092}}]
[debug] QUERY OK db=2.1ms
INSERT INTO "players_players" ("display_name","score","username","inserted_at","updated_at") VALUES ($1,$2,$3,$4,$5) RETURNING "id" ["Evan Czaplicki", 1500, "evancz",
{{2017, 3, 5}, {15, 31, 54, 922251}}, {{2017, 3, 5}, {15, 31, 54, 922256}}]
```

## Saving Our Progress

Since we've made quite a few changes, now would be a good time to run our tests
locally with `mix test`. If everything is passing, let's go ahead and commit our

```shell
$ git add .
$ git commit -m "Update player fields and adjust templates"
```

We don't want to push this to production yet, because we want to handle
authentication first. That will be the topic of our next chapter...
