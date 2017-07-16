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

    channel "score:*", Platform.Web.ScoreChannel
```

Let's go ahead and follow the instructions and add our new channel to the
`user_socket.ex` file. Here's the full `user_socket.ex` file with our new
channel (with most comments removed for brevity's sake):

```elixir
defmodule Platform.Web.UserSocket do
  use Phoenix.Socket

  ## Channels
  channel "score:*", Platform.Web.ScoreChannel

  ## Transports
  transport :websocket, Phoenix.Transports.WebSocket

  def connect(_params, socket) do
    {:ok, socket}
  end

  def id(_socket), do: nil
end
```

We also want to update the `score_channel_test.exs` file that was generated in
the `test/web/channels` folder to include `score:*`.

```elixir
defmodule Platform.Web.ScoreChannelTest do
  use Platform.Web.ChannelCase

  alias Platform.Web.ScoreChannel

  setup do
    {:ok, _, socket} =
      socket("user_id", %{some: :assign})
      |> subscribe_and_join(ScoreChannel, "score:*")

    {:ok, socket: socket}
  end

  test "ping replies with status ok", %{socket: socket} do
    ref = push socket, "ping", %{"hello" => "there"}
    assert_reply ref, :ok, %{"hello" => "there"}
  end

  test "shout broadcasts to score:*", %{socket: socket} do
    push socket, "shout", %{"hello" => "all"}
    assert_broadcast "shout", %{"hello" => "all"}
  end

  test "broadcasts are pushed to the client", %{socket: socket} do
    broadcast_from! socket, "broadcast", %{"some" => "data"}
    assert_push "broadcast", %{"some" => "data"}
  end
end
```

Now, we can go ahead and run our Phoenix tests to make sure everything is still
working as intended:

```shell
$ mix test
Compiling 2 files (.ex)
Generated platform app
......................................

Finished in 0.5 seconds
38 tests, 0 failures
```

## elm-phoenix-socket

The channel features come bundled with the Phoenix framework, but we'll need to
import a new library on the Elm side. To enable communication between the
front-end and back-end, let's use
[elm-phoenix-socket](https://github.com/fbonetti/elm-phoenix-socket).

To get started let's move to the `lib/platform/web/elm` folder in our project
and run the following command to install the package:

```shell
$ elm-package install fbonetti/elm-phoenix-socket
```

This will also import the `elm-lang/websocket` package, and we should see the
following output:

```shell
$ elm-package install fbonetti/elm-phoenix-socket
To install fbonetti/elm-phoenix-socket I would like to add the following
dependency to elm-package.json:

    "fbonetti/elm-phoenix-socket": "2.2.0 <= v < 3.0.0"

May I add that to elm-package.json for you? [Y/n] Y

  Install:
    elm-lang/websocket 1.0.2
    fbonetti/elm-phoenix-socket 2.2.0

Do you approve of this plan? [Y/n] Y
Starting downloads...

  ● elm-lang/websocket 1.0.2
  ● fbonetti/elm-phoenix-socket 2.2.0

Packages configured successfully!
```

## Configuring elm-phoenix-socket

Now, we can work through the
[elm-phoenix-socket README](https://github.com/fbonetti/elm-phoenix-socket/blob/master/README.md)
to configure everything on the Elm side of our application. We'll start by
importing the necessary modules. Let's update the imports at the top of our
`Game.elm` file to include three new `Phoenix` modules:

```elm
import AnimationFrame exposing (diffs)
import Html exposing (Html, div)
import Keyboard exposing (KeyCode, downs, ups)
import Phoenix.Channel
import Phoenix.Push
import Phoenix.Socket
import Random
import Svg exposing (..)
import Svg.Attributes exposing (..)
import Time exposing (Time, every, second)
```

Next, we can add a `phxSocket` field to our model. We'll update our `Model`
type first, and then provide an initial value in our `initialModel` that points
to a new function we'll create called `initPhxSocket`.

```elm
type alias Model =
    { gameState : GameState
    , characterPositionX : Float
    , characterPositionY : Float
    , characterVelocity : Float
    , characterDirection : Direction
    , itemPositionX : Int
    , itemPositionY : Int
    , itemsCollected : Int
    , phxSocket : Phoenix.Socket.Socket Msg
    , playerScore : Int
    , timeRemaining : Int
    }


initialModel : Model
initialModel =
    { gameState = StartScreen
    , characterPositionX = 50.0
    , characterPositionY = 300.0
    , characterVelocity = 0.0
    , characterDirection = Right
    , itemPositionX = 500
    , itemPositionY = 300
    , itemsCollected = 0
    , phxSocket = initPhxSocket
    , playerScore = 0
    , timeRemaining = 10
    }


initPhxSocket : Phoenix.Socket.Socket Msg
initPhxSocket =
    Phoenix.Socket.init "ws://localhost:4000/socket/websocket"
        |> Phoenix.Socket.withDebug
```

In our `initPhxSocket` function, we use the default websocket server for
Phoenix, which is `"ws://localhost:4000/socket/websocket"`. And then we
pipe the results to `Phoenix.Socket.withDebug` so we can take a look at the
DevTools Console once we get things up and running and we'll be able to
inspect the data being passed around.

Before we have a working socket connection, we'll need to add a new message
to our update section. Still following along with the elm-phoenix-socket README
file, we see that we can create a new message type at the bottom to handle
Phoenix socket messages with `PhoenixMsg`.

```elm
type Msg
    = NoOp
    | KeyDown KeyCode
    | KeyUp KeyCode
    | TimeUpdate Time
    | CountdownTimer Time
    | SetNewItemPositionX Int
    | MoveCharacter Time
    | ChangeDirection Time
    | PhoenixMsg (Phoenix.Socket.Msg Msg)
```

And we can add the following case at the bottom of our `update` function:

```elm
PhoenixMsg msg ->
  let
    ( phxSocket, phxCmd ) = Phoenix.Socket.update msg model.phxSocket
  in
    ( { model | phxSocket = phxSocket }
    , Cmd.map PhoenixMsg phxCmd
    )
```

This enables us to track changes in state using the `phxSocket` field in our
model.

Lastly, we can add to our `subscriptions` function to subscribe to changes over
time.

```elm
subscriptions : Model -> Sub Msg
subscriptions model =
    Sub.batch
        [ downs KeyDown
        , ups KeyUp
        , diffs TimeUpdate
        , diffs MoveCharacter
        , diffs ChangeDirection
        , every second CountdownTimer
        , Phoenix.Socket.listen model.phxSocket PhoenixMsg
        ]
```

At this point we should have a working socket connection when we visit the
`http://localhost:4000/elm/game` route in our application. You may need to
restart your local Phoenix server to get things up and running, but if you
load the page and wait a few moment, you should be able to see a "heartbeat"
of websocket messages in the DevTools Console.

If you're not familiar with the Chrome DevTools, you should be able to press
`Command + Option + J` on your keyboard to pull up the JavaScript Console that
displays the messages we're looking for. Another good way to inspect these
is to use the **Network** tab in the Chrome DevTools and click the **WS**
option to only view websocket communication. Here's a screenshot of what it
would look like if you're interested in taking a look:

![Working WebSocket Connection](images/syncing_score_data/working_websocket_connection.png)

For now, our `payload` is empty since we haven't explicitly sent any data yet,
but we can tell that it's working with the `status = "ok"` indicator and see
that it changes over time because it increments the `Just <number>` indicator.
This is what the messages should look like in the DevTools Console:

```shell
Phoenix message: { event = "phx_reply", topic = "phoenix", payload = { status = "ok", response = {} }, ref = Just 0 }
Phoenix message: { event = "phx_reply", topic = "phoenix", payload = { status = "ok", response = {} }, ref = Just 1 }
Phoenix message: { event = "phx_reply", topic = "phoenix", payload = { status = "ok", response = {} }, ref = Just 2 }
```

## Sending Data Over the Socket

Now that we have everything set up, we can start sending data. We want to send
the score that we already have available in the Elm model over the socket to
the Phoenix back-end. We can start by adding a new message to our update
section. Add `SendScore` at the bottom of our `Msg` type:

```elm
type Msg
    = NoOp
    | KeyDown KeyCode
    | KeyUp KeyCode
    | TimeUpdate Time
    | CountdownTimer Time
    | SetNewItemPositionX Int
    | MoveCharacter Time
    | ChangeDirection Time
    | PhoenixMsg (Phoenix.Socket.Msg Msg)
    | SendScore
```

Then, let's go ahead and add the following to the bottom of our `update`
function:

```elm
SendScore ->
    let
        payload =
            Encode.object [ ( "score", Encode.int model.playerScore ) ]
    in
        ( model, Cmd.none ))
```

We'll need to import Elm's JSON encoding package, so add this to the imports
at the top of the file:

```elm
import Json.Encode as Encode
```

With `SendScore`, we're starting to construct a `payload` that we'll use to
send our Elm data. We take the `playerScore` that's part of our Elm model and
we encode is as a JSON integer with `Encode.int`. Then we wrap that up in a
JSON object that we can use to send it to Phoenix. Keep in mind that we're
still using `(model, Cmd.none)` so far, so we're not actually pushing data
over the socket yet.

To continue, we'll use the `Phoenix.Push` module that we imported at the top of
our file. We want to initialize using the `"score:*"` channel that we
created on the Phoenix side, and we'll use the `payload` we constructed to send
along the relevant data. We'll also use the pipe operator to pass data along
and handle the success and failure cases.

```elm
SendScore ->
    let
        payload =
            Encode.object [ ( "score", Encode.int model.playerScore ) ]

        phxPush =
            Phoenix.Push.init "shout" "score:*"
                |> Phoenix.Push.withPayload payload
                |> Phoenix.Push.onOk ReceiveScore
                |> Phoenix.Push.onError HandleSendError
    in
        ( model, Cmd.none )
```

We'll need to scaffold out new messages for `ReceiveScore` and
`HandleSendError` for the success and failure cases, respectively. We can add
these to our `Msg` type first, and they'll both take an `Encode.Value` as an
argument:

```elm
type Msg
    = NoOp
    | KeyDown KeyCode
    | KeyUp KeyCode
    | TimeUpdate Time
    | CountdownTimer Time
    | SetNewItemPositionX Int
    | MoveCharacter Time
    | ChangeDirection Time
    | PhoenixMsg (Phoenix.Socket.Msg Msg)
    | SendScore
    | ReceiveScore Encode.Value
    | HandleSendError Encode.Value
```

And we can add cases at the bottom of our `update` function to get our code
back to a state with no errors, and we're one step closer to connecting our
front-end with our back-end.

```elm
ReceiveScore _ ->
    ( model, Cmd.none )

HandleSendError _ ->
    Debug.log "Error sending score over socket."
        ( model, Cmd.none )
```

## Executing the Push

In our `SendScore` case, now we're going to tie everything together by sending
data over the `phxSocket`.

```elm
SendScore ->
    let
        payload =
            Encode.object [ ( "score", Encode.int model.playerScore ) ]

        phxPush =
            Phoenix.Push.init "shout" "score:*"
                |> Phoenix.Push.withPayload payload
                |> Phoenix.Push.onOk ReceiveScore
                |> Phoenix.Push.onError HandleSendError

        ( phxSocket, phxCmd ) =
            Phoenix.Socket.push phxPush model.phxSocket
    in
        ( { model | phxSocket = phxSocket }
        , Cmd.map PhoenixMsg phxCmd
        )
```

Now we can go back up to the `initPhxSocket` function we created at the top,
and pipe things along to the `"score:*"` channel with `ReceiveScore`.

```elm
initPhxSocket : Phoenix.Socket.Socket Msg
initPhxSocket =
    Phoenix.Socket.init "ws://localhost:4000/socket/websocket"
        |> Phoenix.Socket.withDebug
        |> Phoenix.Socket.on "shout" "score:*" ReceiveScore
```

## Triggering SendScore

Finally, we just need to trigger the `SendScore` message whenever we want to
send our score over the socket. We'll find a good time to continuously update
our score, but for now we want to set up a manual trigger so we can test things
out. We'll set it up so that we can _click_ the score text with our mouse, and
check the DevTools console to view the `payload` that's being sent over the
socket.

At the top of our file, let's import the `onClick` functionality from
`Html.Events`.

```elm
import Html.Events exposing (onClick)
```

Now we can update our `viewGameScore` function to trigger the `SendScore`
message when we click that area in our SVG.

```elm
viewGameScore : Model -> Svg Msg
viewGameScore model =
    let
        currentScore =
            model.playerScore
                |> toString
                |> String.padLeft 5 '0'
    in
        Svg.svg [ onClick SendScore ]
            [ viewGameText 25 25 "SCORE"
            , viewGameText 25 40 currentScore
            ]
```

It looks like clicking the score area successfully triggers `SendScore`, but
we've got an issue with the way we set up the channel. Our topic is set to
`score:*`, but we're getting an `"error"` status and it says the reason is an
`"unmatched topic"`.

