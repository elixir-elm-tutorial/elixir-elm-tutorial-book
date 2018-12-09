# Phoenix Channel Setup

We have our game platform up and running, where users can sign in and play a
simple Elm game that tracks a score. Now, let's work towards syncing the Elm
front-end of our application with the Phoenix back-end. We'll learn about
Phoenix channels with the goal of being able to communicate the score from
games back to a player's account in real-time.

Since this chapter is fairly tedious in terms of learning about channels and
configuring both the back-end and the front-end, here's a brief outline of how
we'll approach the topics:

- Introduction to Phoenix channels and WebSockets
- Creating our first Phoenix channel on the back-end
- Setting up the Phoenix channel for joining and handling messages
- Configuring the Elm front-end to work with channels
- Joining the channel and pushing data over the socket

## Warning

Now that we're working at the intersection of Elm and Phoenix, it's good to
keep in mind that things are still rapidly changing in the community. There's
no "official" client to hook up Elm and Phoenix yet. Although the libraries
and techniques tend to change, it's also good to keep in mind that the overall
concepts in Elixir and Elm both tend to stay the same. In other words, the
material in this chapter is subject to change as new integration techniques
become standard practice, but the concepts we're using to build our application
will still be useful even as things change.

## Channels

Essentially, Phoenix channels give us a way to send and receive messages. A
chatroom application is a common example of how channels work. When users enter
the chatroom, they "join" the channel. Then, we can "broadcast" the chat
messages to all users that have joined the channel.

We're going to use Phoenix channels for a similar purpose. We're going to take
the `playerScore` field that we're tracking in our Elm application, and we're
going to broadcast that value to other users so all players can see scores for
a particular game being tracked in real-time. We'll also take a look at how to
store the scores as records in our database using the `gameplays` table we
configured earlier in the book.

## Socket Connections

A helpful way to think about Phoenix channels is to think of data being sent
back and forth over the socket. We saw something similar in this book while
working with the Phoenix framework. Clients make a _request_ and the server
returns a _response_, and Phoenix uses a `conn` to store all the data that gets
communicated back and forth.

Channels use the same idea, but instead of a `conn` we use a `socket`
connection. We can assign data to the socket, and then it's available on both
the front-end and back-end without needing to make additional HTTP requests.

## Score Channel

To get started, we can create a file for our new channel in the
`lib/platform_web/channels` folder. Let's use `score_channel.ex` as the
filename, and add the following content:

```elixir
defmodule PlatformWeb.ScoreChannel do
  use PlatformWeb, :channel
end
```

To get our new channel up and running, let's open the `user_socket.ex` file
in the same `lib/platform_web/channels` folder. At the top, we'll see comments
that show how to add a new channel.

```elixir
defmodule PlatformWeb.UserSocket do
  use Phoenix.Socket

  ## Channels
  # channel "room:*", PlatformWeb.RoomChannel

  # ...
end
```

We'll replace the commented example with the channel we're going to be working
with. We're setting things up to work with the `PlatformWeb.ScoreChannel` that
we just created. The `"score:*"` part refers to "topic" and "subtopic". That
means we can use `"score:platformer"` to track scores for our Platformer game,
or something like `"score:racer"` to track scores for a different game. The
`"*"` character means that we're joining the `"score"` topic and matching _any_
subtopics.

```elixir
defmodule PlatformWeb.UserSocket do
  use Phoenix.Socket

  ## Channels
  channel "score:*", PlatformWeb.ScoreChannel

  # ...
end
```

## Joining the Channel

Let's add a `join/3` function to the `score_channel.ex` file. This function
will take three arguments:

- **The Topic:** We're joining `"score:platformer"` to track scores for our
  Platformer game.
- **The Payload:** We're using `_payload` to ignore this for now, but this will
  contain the data that we pass over the socket.
- **The Socket:** This is the WebSocket connection that we return in the body of
  the function.

```elixir
defmodule PlatformWeb.ScoreChannel do
  use PlatformWeb, :channel

  def join("score:platformer", _payload, socket) do
    {:ok, socket}
  end
end
```

Now, let's add another function that will allow us to handle incoming messages
from the client. We'll use the `handle_in/3` function, and we'll listen for a
`"save_score"` message to trigger it.

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

This will allow us to listen for a `"save_score"` message that we'll send from
our Elm client. Inside the `handle_in/3` function, we use the
[`broadcast/3`](https://hexdocs.pm/phoenix/Phoenix.Channel.html#broadcast/3)
function, which will relay the results to all players on the channel. We'll
make some additional changes to this `handle_in/3` function in the next chapter
too, but for now all we need to know is that players will be able to join our
channel and then send `"save_score"` messages to broadcast their score to all
other players connected to the socket (and any other messages that don't match
our `"save_score"` name will be ignored automatically).

Before we move back to our Elm application, let's enable some settings in our
Phoenix endpoint to ensure everything we need is enabled. Open the
`lib/platform_web/endpoint.ex` file and set both the `websocket` and `longpoll`
settings to a value of `true`:

```elixir
defmodule PlatformWeb.Endpoint do
  # ...

  socket "/socket", PlatformWeb.UserSocket,
    websocket: true,
    longpoll: true

  # ...
end
```

We haven't configured our front-end to work with the channel yet, but we've
managed to take care of the initial channel setup on the back-end.

## Elm and Phoenix Channels

While channel features come bundled with the Phoenix framework, we'll still
need to import a new package on the Elm side to enable communication between
the front-end and back-end.

To get started, let's move to the `assets/elm` folder in our project and run
the following command to install the
[`slashmili/phoenix-socket`](https://package.elm-lang.org/packages/slashmili/phoenix-socket/latest)
package:

```shell
$ elm install slashmili/phoenix-socket
```

In the near future, we'll likely have different options for how we want to
connect Elm and Phoenix. And it's also likely that a standard approach will
develop over time. We're using `slashmili/phoenix-socket` in this book because
it works well with the latest version of Elm, but if you're interested in
pursuing your own approach and pushing the boundaries, be sure to check out the
prior art that's available in packages like `fbonetti/elm-phoenix-socket` and
`saschatimme/elm-phoenix`.

With this package installed, we can move on and update our Elm front-end
application to send messages over the channel to the Phoenix back-end.

## Configuring Elm with Phoenix Channels

We've already imported a handful of packages at the top of our `Platformer.elm`
file, and now we'll need to import a handful of new ones to work with Phoenix
channels. Let's update the top of `Platformer.elm` with the following:

```elm
module Games.Platformer exposing (main)

import Browser
import Browser.Events
import Html exposing (Html, div)
import Json.Decode as Decode
import Json.Encode as Encode
import Phoenix
import Phoenix.Channel
import Phoenix.Message
import Phoenix.Push
import Phoenix.Socket
import Random
import Svg exposing (..)
import Svg.Attributes exposing (..)
import Time
```

We're importing several new modules here. We'll use `Json.Encode` to encode our
player score data as JSON before we send it to the back-end. We're also
importing a total of five different `Phoenix` modules from the
`slashmili/phoenix-socket` package, and we'll use each of them throughout the
rest of this chapter.

Next, let's can add a `phxSocket` field to our model. We'll update our `Model`
type first, and then provide an initial value in our `initialModel` that points
to a new function we'll call `initialSocket`.

```elm
type alias Model =
    { characterDirection : Direction
    , characterPositionX : Int
    , characterPositionY : Int
    , gameState : GameState
    , itemPositionX : Int
    , itemPositionY : Int
    , itemsCollected : Int
    , phxSocket : Phoenix.Socket.Socket Msg
    , playerScore : Int
    , timeRemaining : Int
    }


initialModel : Model
initialModel =
    { characterDirection = Right
    , characterPositionX = 50
    , characterPositionY = 300
    , gameState = StartScreen
    , itemPositionX = 500
    , itemPositionY = 300
    , itemsCollected = 0
    , phxSocket = initialSocket
    , playerScore = 0
    , timeRemaining = 10
    }
```

Here's the new `initialSocket` function we can use to initialize the socket
connection with `Phoenix.Socket.init`. When our Phoenix server is running,
we'll be able to use the `devSocketServer` value for the WebSocket connection
that's being served as the default from Phoenix.

```elm
initialSocket : Phoenix.Socket.Socket Msg
initialSocket =
    let
        devSocketServer =
            "ws://localhost:4000/socket/websocket"
    in
    Phoenix.Socket.init devSocketServer
```

This is a good start because it means we're already initializing the socket
connection as soon as our Elm application loads. But we'll have to take a few
more steps before we're able to send messages.

## The Update Function

Next, let's update our `Msg` type first to handle Phoenix socket messages with
`PhoenixMsg`.

```elm
type Msg
    = CountdownTimer Time.Posix
    | GameLoop Float
    | KeyDown String
    | NoOp
    | PhoenixMsg (Phoenix.Message.Msg Msg)
    | SetNewItemPositionX Int
```

And we can add the following inside the `case` expression of our `update`
function:

```elm
update : Msg -> Model -> ( Model, Cmd Msg )
update msg model =
    case msg of
        -- ...

        PhoenixMsg phxMsg ->
            let
                ( updatedSocket, updatedCmd ) =
                    Phoenix.update PhoenixMsg phxMsg model.phxSocket
            in
            ( { model | phxSocket = updatedSocket }, updatedCommand )

        -- ...
```

This enables us to track changes in state using the `phxSocket` field in our
model.

## Socket Subscription

Lastly, we can add to our `subscriptions` function to subscribe to changes over
time.

```elm
subscriptions : Model -> Sub Msg
subscriptions model =
    Sub.batch
        [ Browser.Events.onKeyDown (Decode.map KeyDown keyDecoder)
        , Browser.Events.onAnimationFrameDelta GameLoop
        , Time.every 1000 CountdownTimer
        , Phoenix.listen PhoenixMsg model.phxSocket
        ]
```

At this point, we should have a working socket connection when we visit
`http://localhost:4000/games/platformer` in our browser. You may need to
restart your local Phoenix server to get things up and running, but if you
load the page and wait a few moments, you should be able to see something like
the following in the server server console:

```shell
[info] GET /games/platformer
[info] Sent 200 in 5ms
[info] CONNECT PlatformWeb.UserSocket
  Transport: :longpoll
  Connect Info: %{}
  Parameters: %{}
[info] Replied PlatformWeb.UserSocket :ok
```

## Sending Data Over the Socket

Our goal for this chapter is to send data over the socket. We want to send the
score that we already have available in the Elm model over the socket to the
Phoenix back-end. It'll be a two-step process of joining the channel first and
then sending the data, so let's set up our `Msg` type and `update` function.

First, add `SaveScore` to our `Msg` type:

```elm
type Msg
    = CountdownTimer Time.Posix
    | GameLoop Float
    | KeyDown String
    | NoOp
    | PhoenixMsg (Phoenix.Message.Msg Msg)
    | SaveScore
    | SetNewItemPositionX Int
```

Then, let's add the following to the `update` function:

```elm
SaveScore ->
    let
        payload =
            Encode.object [ ( "player_score", Encode.int model.playerScore ) ]
    in
    ( model, Cmd.none )
```

With `SaveScore`, we're starting to construct a `payload` that we'll use to
send our Elm data. We take the `playerScore` that's part of our Elm model, and
we encode it as a JSON integer with `Encode.int`. Then, we wrap that up in a
JSON object that we can use to send it to the Phoenix back-end. Keep in mind
that we're still using `(model, Cmd.none)` so far, so we're not actually
pushing data over the socket yet.

## Phoenix.Push

To continue, we'll use the `Phoenix.Push` module that we imported at the top of
our file. We want to initialize using the `"score:platformer"` channel that we
created on the Phoenix side, and we'll use the `payload` we constructed to send
along the relevant data. We'll also use the pipe operator to pass data along
and handle the success and failure cases.

```elm
SaveScore ->
    let
        payload =
            Encode.object [ ( "player_score", Encode.int model.playerScore ) ]

        phxPush =
            Phoenix.Push.init "save_score" "score:platformer"
                |> Phoenix.Push.withPayload payload
                |> Phoenix.Push.onOk SaveScoreSuccess
                |> Phoenix.Push.onError SaveScoreError
    in
    ( model, Cmd.none )
```

We'll need to scaffold out new messages for `SaveScoreSuccess` and
`SaveScoreError` for the success and failure cases, respectively. We can add
these to our `Msg` type first, and they'll both take an `Encode.Value` as an
argument:

```elm
type Msg
    = CountdownTimer Time.Posix
    | GameLoop Float
    | KeyDown String
    | NoOp
    | SaveScoreSuccess Encode.Value
    | SaveScoreError Encode.Value
    | SaveScore
    | PhoenixMsg (Phoenix.Message.Msg Msg)
    | SetNewItemPositionX Int
```

And we can add cases at the bottom of our `update` function to get our code
back to a state with no errors, and we're one step closer to connecting our
front-end with our back-end.

```elm
SaveScoreSuccess value ->
    Debug.log "Successfully sent score over socket."
        ( model, Cmd.none )

SaveScoreError message ->
    Debug.log "Error sending score over socket."
        ( model, Cmd.none )
```

## Finishing the Push Setup

In our `SaveScore` case, we're going to connect everything together by sending
data over the `phxSocket`.

```elm
SaveScore ->
    let
        payload =
            Encode.object [ ( "player_score", Encode.int model.playerScore ) ]

        phxPush =
            Phoenix.Push.init "save_score" "score:platformer"
                |> Phoenix.Push.withPayload payload
                |> Phoenix.Push.onOk SaveScoreSuccess
                |> Phoenix.Push.onError SaveScoreError

        ( updatedSocket, updatedCommand ) =
            Phoenix.push PhoenixMsg phxPush model.phxSocket
    in
    ( { model | phxSocket = updatedSocket }, updatedCommand)
```

## Joining the Score Channel

Now that we have everything in place to push our data over the socket, we'll
just need to join the `"score:platformer"` channel and broadcast the player's
score. We'll accomplish this in two steps. First, we'll create a button that
allows players to join the channel. Then, we'll add another button that enables
us to save scores.

Below our `initialSocket` function, let's start by creating a new function
called `initialChannel`.

```elm
initialChannel : Phoenix.Channel.Channel Msg
initialChannel =
    "score:platformer"
        |> Phoenix.Channel.init
        |> Phoenix.Channel.on "save_score" SaveScoreSuccess
```

Next, we'll update our `Msg` type with a new `JoinChannel` message:

```elm
type Msg
    = CountdownTimer Time.Posix
    | GameLoop Float
    | JoinChannel
    | KeyDown String
    | NoOp
    | SaveScoreSuccess Encode.Value
    | SaveScoreError Encode.Value
    | SaveScore
    | PhoenixMsg (Phoenix.Message.Msg Msg)
    | SetNewItemPositionX Int
```

With our `initialChannel` and `JoinChannel` code in place, we can set up our
`update` function. For the `JoinChannel` case, we're taking the existing socket
connection with `model.phxSocket` and using `Phoenix.join` with our
`initialChannel` function to join the channel and update it in the model.

```elm
update : Msg -> Model -> ( Model, Cmd Msg )
update msg model =
    case msg of
        -- ...

        JoinChannel ->
            let
                ( updatedSocket, updatedCommand ) =
                    Phoenix.join PhoenixMsg initialChannel model.phxSocket
            in
            ( { model | phxSocket = updatedSocket }, updatedCommand )

        -- ...
```

## Triggering JoinChannel and SaveScore

Finally, we just need to trigger the `JoinChannel` and `SaveScore` messages
to send our score over the socket. We'll set it up so that we can click a
button and then check the server console to view the `payload` that's being
sent over the socket.

At the top of our file, let's import the `onClick` functionality from
`Html.Events`. While we're here, we also want to make a quick change to the
`Html` import so we can use the `button` element. Here are the `Html` imports:

```elm
import Html exposing (Html, button, div)
import Html.Events exposing (onClick)
```

## Adding a Button to the View

Now, we can create new `viewJoinChannelButton` and `viewSaveScoreButton`
functions and add them to our main `view` to trigger the `JoinChannel` and
`SaveScore` messages.

```elm
view : Model -> Html Msg
view model =
    div []
        [ viewGame model
        , viewJoinChannelButton
        , viewSaveScoreButton
        ]


viewJoinChannelButton : Html Msg
viewJoinChannelButton =
    div []
        [ button [ onClick JoinChannel, class "button" ]
            [ text "Join Channel" ]
        ]


viewSaveScoreButton : Html Msg
viewSaveScoreButton =
    div []
        [ button [ onClick SaveScore, class "button" ]
            [ text "Save Score" ]
        ]
```

## Testing Out the Socket

We've got everything configured, and we should be able to test out our working
socket payload. Here are the steps to try it:

- Start the server with `mix phx.server` and load the game in the browser.
- Play the game and collect a few coins to increment the player's score.
- Click the "Join Channel" button.
- Click the "Save Score" button.

Although we won't see any changes in the UI, we should be able to check the
server console and see things working.

When the game loads on the page we see that we connect to the socket:

```shell
[info] GET /games/platformer
[info] CONNECT PlatformWeb.UserSocket
[info] Replied PlatformWeb.UserSocket :ok
```

Then, when we click the "Join Channel" button we see the successful join
message:

```shell
[info] JOIN "score:platformer" to PlatformWeb.ScoreChannel
[info] Replied score:platformer :ok
```

Lastly, when we click the "Save Score" button we see the data being sent over
the socket:

```shell
[debug] INCOMING "save_score" on "score:platformer" to PlatformWeb.ScoreChannel
  Parameters: %{"player_score" => 300}
```

When we trigger the `"save_score"` event, we're broadcasting the data over the
socket. The broadcast allows us to send the data in real-time to any other
players connected to the socket. In the example above, you can see that we
collected three coins for a score of `300`, and that is reflected in the
payload with `%{"player_score" => 300}`.

## Summary

We made it a _long_ way in this chapter. We managed to take care of most of the
hard work, and we're successfully sending data from the Elm front-end to the
Phoenix back-end. We created a new Phoenix channel, installed a package, and
configured our game to send a payload. But we're not displaying the player
scores in our application or persisting the data anywhere yet. We'll take a
look at these topics in the next chapter.
