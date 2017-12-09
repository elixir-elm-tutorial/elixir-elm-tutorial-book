# Phoenix Channels

We created our `ScoreChannel` and set up elm-phoenix-socket in the last
chapter, but we haven't really worked with Phoenix channels much outside of the
initial configuration. In this chapter, we'll take that data from our Elm
front-end game, and find a good way to handle it with our Phoenix back-end.

## Handling a New Message

TODO: Elm side – Change "shout" to "sync_score".

TODO: Elixir channel – Add aliases

```elixir
alias Platform.Repo
alias Platform.Accounts.Player
```

TODO: Elixir channel – implementation

```elixir
  # Sync score.
  def handle_in("sync_score", payload, socket) do
    IO.puts "HIIIIII"

    IO.puts "Payload Score"
    IO.puts payload["score"]

    player = Repo.get!(Player, 4)
    IO.puts "Player Username"
    IO.puts player.username

    IO.puts "Player Score"
    IO.puts player.score || "0"

    player = %{player | score: payload["score"]}
    IO.puts "New Score?"
    IO.puts player.score || "0"

    broadcast socket, "sync_score", %{score: payload["score"]}
    {:noreply, socket}
  end
```
