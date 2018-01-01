# Syncing Score Data

We created our `ScoreChannel` and set up elm-phoenix-socket in the last
chapter, but we haven't really worked with Phoenix channels much outside of the
initial configuration. In this chapter, we'll take that data from our Elm
front-end game, and find a good way to handle it with our Phoenix back-end.

## Planning Out Our Approach

Our game is tracking a `playerScore` field on the Elm side as the character
collects items. And we managed to configure elm-phoenix-socket in the last
chapter, so we have the score being sent over the `ScoreChannel` as the
`payload` value. But we'd like to display player score updates in real-time,
and be able to save scores to the database as `Gameplay` records.

We'll use the "Save Score" button to save our player scores to the database,
and to broadcast score changes to all players connected to the socket.
When we create new new database records for the score, we'll need to include
the `game_id`, the `player_id`, and the `player_score` values.

Then, we can work towards displaying recent player scores below the game.

## Updating our ScoreChannel

Let's make some changes to our `lib/platform_web/channels/score_channel.ex`
file. This is what we have so far:

```elixir
defmodule PlatformWeb.ScoreChannel do
  use PlatformWeb, :channel

  def join("score:platformer", _payload, socket) do
    {:ok, socket}
  end

  def handle_in("save_score", payload, socket) do
    broadcast(socket, "save_score", payload)
    {:noreply, socket}
  end
end
```

Instead of manually coding the name of the game in our topic, we'll destructure
the slug by pattern matching with the `<>` string concatenation operator.

```elixir
def join("score:" <> game_slug, _payload, socket) do
  {:ok, socket}
end
```

Our `Platformer.elm` file is already set up to join the `"score:platformer"`
topic. But this change means that we'll be able to use this same channel for
other games in the future, and the slug field will allow us to differentiate
between the games.

The reason this is important is that we need to know which game is being
played and the current player in order to save their score.

## Tracking Data Over the Socket

Let's make some additional changes to the `join/3` function we worked with in
the last section. We're going to learn how to assign values to the `socket`
that we can use to work with data in our channel.

Now, we can find the game that's currently being played, and assign the
`game_id` as a value we can work with in the socket. In our `join/3` function,
let's make the following changes:

```elixir
def join("score:" <> game_slug, _payload, socket) do
  game = Platform.Products.get_game_by_slug!(game_slug)
  socket = assign(socket, :game_id, game.id)
  {:ok, socket}
end
```

The `Products.get_game_by_slug!/1` function is something we created in the
**Game Setup** chapter. We can use it here to find the game record, and then
find the `game.id`.

Then, we use the `assign(socket, :game_id, game.id)` syntax to assign that
value and make it accessible when working with our `socket`. In other words,
we'll be able to use `socket.assigns.game_id` to access this value in any of
our handle functions now. This will be useful in the next section.

## Creating the Payload

In order to save our score records to the database, we'll also need to grab the
`player_score` value. Let's deconstruct that value from the `payload` argument
in our `handle_in/3` function.

```elixir
def handle_in("save_score", %{"player_score" => player_score} = payload, socket) do
  broadcast(socket, "save_score", payload)
  {:noreply, socket}
end
```

We're pattern matching the `player_score` from the `payload` so the value is
accessible inside our function.

Now, let's construct our `payload` and create our new `Gameplay` record inside
this function. We have the `player_score` value available, and the `game_id` is
accessible in the `socket`.

We're also going to hard-code the `player_id` value (using `3` for the
`chrismccord` account we've been seeing in all the screenshots) for now. We'll
take a look at socket authentication and tracking the current user in our
channel soon.

```elixir
def handle_in("save_score", %{"player_score" => player_score} = payload, socket) do
  payload = %{
    player_score: player_score,
    game_id: socket.assigns.game_id,
    player_id: 3
  }

  Platform.Products.create_gameplay(payload)
  broadcast(socket, "save_score", payload)
  {:noreply, socket}
end
```

## Creating Gameplays

Our channel features should now work using the `create_gameplay/1` function
from our `lib/platform/products.ex` file. To save new records in our
`"gameplays"` table, we'll be passing a `game_id`, a `player_id`, and a
`player_score` to `create_gameplay/1`.

```elixir
def create_gameplay(attrs \\ %{}) do
  %Gameplay{}
  |> Gameplay.changeset(attrs)
  |> Repo.insert()
end
```

When we load our game in the browser, we should now be able to click the "Save
Score" button and create records in the database.

For the output and screenshot below, we collected a few coins and then clicked
the "Save Score" button. We can see that the `"save_score"` message was
triggered for the `"score:platformer"` topic. And the `payload` contains the
data we set up in the `handle_in/3` function from our `ScoreChannel`.

We have the `player_score` field being tracked from the game, and we're taking
the `game_id` from the value we grabbed in the `join/3` function. We can also
see the hard-coded value of `3` that we set for the `player_id` field, and
we'll take care of socket authentication soon.

```shell
Phoenix message: { event = "save_score", topic = "score:platformer", payload = { player_score = 300, player_id = 3, game_id = 1 }, ref = Nothing }
```

![Working Channel Payload](images/syncing_score_data/working_channel_payload.png)

## Viewing Gameplay Records

At this point, we should have a working button to save our player scores to the
database. But we don't have any way of viewing them yet, and we're currently
hard-coding the `player_id` value.

...
