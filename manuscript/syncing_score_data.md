# Syncing Score Data

We have our game platform up and running where users can log in and play a
simple Elm game that tracks a score. Now let's work towards syncing the Elm
front-end of our application with the Phoenix back-end. We'll learn about
Phoenix channels with the goal of being able to communicate the score from
games back to a player's account in real-time.


## Phoenix Channels

Let's get started by using the Phoenix generator to create a new channel. Open
up your Terminal so we can run the following shell command:

```shell
mix phx.gen.channel Score
```

This command will create a `lib/platform/web/channels/score_channel.ex` file
along with a test file for us too.

The idea is that Phoenix channels will allow us to communicate over a WebSocket
so that we can sync our `playerScore` field from our Elm application to the
player's `score` field in our Phoenix application.

When we run the generator mentioned above, we should see the following output:

```shell
$ mix phx.gen.channel Score
* creating web/channels/score_channel.ex
* creating test/channels/score_channel_test.exs

Add the channel to your `web/channels/user_socket.ex` handler, for example:

    channel "score:lobby", Platform.ScoreChannel
```

Let's go ahead and follow the instructions and add our new channel to the
`user_socket.ex` file. Here's the full `user_socket.ex` file with our new
channel (with most comments removed for brevity's sake):

```elixir
defmodule Platform.Web.UserSocket do
  use Phoenix.Socket

  ## Channels
  channel "score:lobby", Platform.ScoreChannel

  ## Transports
  transport :websocket, Phoenix.Transports.WebSocket

  def connect(_params, socket) do
    {:ok, socket}
  end
1
  def id(_socket), do: nil
end
```

We can also go ahead and run our Phoenix tests to make sure everything is still
working as intended:

```shell
$ mix test
Compiling 2 files (.ex)
Generated platform app
......................................

Finished in 0.5 seconds
38 tests, 0 failures
```



## TODO

- Create Phoenix channel
- Import elm-phoenix-socket
