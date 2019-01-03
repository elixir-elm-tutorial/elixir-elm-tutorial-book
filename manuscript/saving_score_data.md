# Saving Score Data

In this chapter, we'll work towards giving players the ability to save their
scores to the database. We've already covered many of the concepts we need to
get this done, so this chapter will be a good refresher and an opportunity to
practice setting up a new message within our Phoenix channel.

We'll also tie together a lot of the concepts from both the Phoenix back-end
and the Elm front-end so we can both save new data to the database and fetch
existing data to work with.

## Saving Scores with Our ScoreChannel

We currently have a `ScoreChannel` module that listens for incoming messages,
and so far we've set up a message called `"broadcast_score"` that allows
players to broadcast their scores to any other players connected to the socket
in real-time. But what if we want to give users an option to save their scores
instead of seeing them disappear when the page reloads?

Let's open up the `lib/platform_web/channels/score_channel.ex` file and start
listening for a new message called `"save_score"`.

```elixir
defmodule PlatformWeb.ScoreChannel do
  # ...

  def handle_in("save_score", payload, socket) do
    IO.inspect(payload, label: "Saving the score payload to the database")
    # Save player scores to the database
    {:noreply, socket}
  end
end
```

Back in the **Phoenix API** chapter earlier in the book, we set up `gameplays`
to create the relationship between players and scores. When we create a new
`gameplay`, it needs to contain the following fields:

- `game_id` (which references the `games` table)
- `player_id` (which references the `players` table)
- `player_score` (which stores the player's score as an integer)

In the last chapter, we also worked with this same data to create a `Gameplay`
type in our Elm application too. As a reminder, here's the code we used for the
`"broadcast_score"` message:

```elixir
# Broadcast for authenticated players
def handle_in(
      "broadcast_score",
      %{"player_score" => player_score} = payload,
      %{assigns: %{game_id: game_id, player_id: player_id}} = socket
    ) do
  payload = %{
    game_id: game_id,
    player_id: player_id,
    player_score: player_score
  }

  IO.inspect(payload, label: "Broadcasting the score payload over the channel")
  broadcast(socket, "broadcast_score", payload)
  {:noreply, socket}
end
```

Let's take the same approach for our `"save_score"` message. We're basically
going to have all the same code for the arguments and constructing the
`payload`. The only major difference is that instead of calling the
`broadcast/3` function we're going to make a call to the
`Platform.Products.create_gameplay/1` function we created back in the
**Phoenix API** chapter. Let's go ahead and replace the `handle_in/3` function
we created above with the following:

```elixir
# Save scores for authenticated players
def handle_in(
      "save_score",
      %{"player_score" => player_score} = payload,
      %{assigns: %{game_id: game_id, player_id: player_id}} = socket
    ) do
  payload = %{
    game_id: game_id,
    player_id: player_id,
    player_score: player_score
  }

  IO.inspect(payload, label: "Saving the score payload to the database")
  Platform.Products.create_gameplay(payload)
  {:noreply, socket}
end
```

This might seem like a lot of code, but it's the exact same approach we took in
the last chapter. We're pattern matching all the relevant values we need in the
arguments to the `handle_in/3` function and then using those values to create a
new `payload`.

The key line of code here is that we're sending that `payload` to the
`Platform.Products.create_gameplay/1` function, which will store the gameplays
in the database for us.