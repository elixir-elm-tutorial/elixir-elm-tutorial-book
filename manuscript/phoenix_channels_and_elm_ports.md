# Phoenix Channels and Elm Ports

We have our game platform up and running, where users can sign in and play a
simple Elm game that tracks a score. Now, we'll be working at the intersection
of Elixir, Elm, and JavaScript to sync the data between the Elm front-end and
the Phoenix back-end.

We'll learn about Phoenix channels with the goal of being able to communicate
the score from our game to other players using WebSockets as well as saving the
scores to the database and display them on the screen. We'll also get an
introduction to Elm ports, which allow us to communicate between Elm and
JavaScript while retaining the typesafe benefits of working with Elm.

## A Brief Warning

It's good to keep in mind that the communities for Elixir and Elm are rapidly
evolving, and the approach for integrating these languages can change quickly
as well.

There will likely be a time in the near future where packages make it very
simple for developers to integrate the two languages with a straightforward
API, but currently it can be a tedious process to connect things properly.

The good news is that this knowledge is very valuable, because any non-trivial
Elm application is likely going to need to work with JavaScript code and
knowing how to use ports is essential.

## Introduction to Channels

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
`"broadcast_score"` message to trigger it.

```elixir
defmodule PlatformWeb.ScoreChannel do
  use PlatformWeb, :channel

  def join("score:platformer", _payload, socket) do
    {:ok, socket}
  end

  def handle_in("broadcast_score", payload, socket) do
    broadcast(socket, "broadcast_score", payload)
    {:noreply, socket}
  end
end
```

This will allow us to listen for a `"broadcast_score"` message that we'll send
from our Elm client. Inside the `handle_in/3` function, we use the
[`broadcast/3`](https://hexdocs.pm/phoenix/Phoenix.Channel.html#broadcast/3)
function, which will relay the results to all players on the channel. We'll
make some additional changes to this `handle_in/3` function in the next chapter
too, but for now all we need to know is that players will be able to join our
channel and then send `"broadcast_score"` messages to broadcast their score to
all other players connected to the socket (and any other messages that don't
match our `"broadcast_score"` name will be ignored automatically).

We haven't configured our front-end to work with the channel yet, but we've
managed to take care of the initial channel setup on the back-end.

## Introduction to Elm Ports

Now that we've configured our Phoenix channel feature, we need to figure out
how to get data from our Elm front-end application to the back-end. To
accomplish this, we'll use Elm ports.

The official Elm guide has a
[ports chapter](https://guide.elm-lang.org/interop/ports.html) if you're
interested in reading it as a quick introduction. For our purposes, all you
need to know is that ports give us a way to send data out of our Elm
application (the player score) and to receive data back into our Elm
application (data from Phoenix).

One thing to keep in mind is that ports don't communicate directly from Elm to
Elixir. Ports allow for interoperability between Elm and JavaScript. That means
JavaScript will be the glue between our Elm front-end and our Phoenix back-end.

In other words, we'll continue writing our Elm code in the
`assets/elm/src/Games/Platformer.elm` file. And we'll continue writing Elixir
code for our channel in `lib/platform_web/channels/score_channel.ex`. And we'll
use the `assets/js/app.js` (we worked with this file in previous chapters when
we initialized our Elm application) to tie everything together.

You might be surprised at how easy it is to set up Elm ports at first. But one
more thing to note is that Elm is a strongly typed language where we're
deliberate about the types of values we work with. For instance, our
`playerScore` field is an `Int` type in our Elm application. As we work with
JavaScript code, we'll need to encode and decode the values properly.

## Configuring Elm Ports

The first step we need to take is to update the first line of code in our
`Platformer.elm` file. We're going to add `port` at the very beginning to
indicate that this module communicates with the outside world.

```elm
port module Games.Platformer exposing (main)
```

Let's also take this opportunity to update the imports at the top of our file
as well. We already have the `Json.Decode` package imported, and now let's also
include the `Json.Encode` package (which we'll alias to `Encode`) so we can
send JSON data out of our Elm application.

We're going to create a button to trigger our port too, so let's add a couple
of `Html` imports as well.

```elm
import Browser
import Browser.Events
import Html exposing (Html, button, div)
import Html.Attributes
import Html.Events
import Json.Decode as Decode
import Json.Encode as Encode
import Random
import Svg exposing (..)
import Svg.Attributes exposing (..)
import Time
```

Before we worry about receiving data from outside our Elm application, let's
start by creating a port to send data from Elm to JavaScript.

We're going to call our first port `broadcastScore`, and our goal will be to
take the `playerScore` field from our model and encode it as JSON when we
send it out to JavaScript (which we'll later handle with Phoenix too).

```elm
-- PORTS

port broadcastScore : Encode.Value -> Cmd msg
```

Bear in mind that you can put this code wherever you like in the
`Platformer.elm` file. But I like to put the code for my ports just below the
`subscriptions` function since we tend to subscribe to outside data as well.

You'll notice that the syntax for our `broadcastScore` is pretty simple. We're
basically just saying that we'll take a JSON encoded value and create a command
with it, which we can then trigger with an update message.

## Triggering the Port in the Update

Next, let's update our `Msg` type with a new `BroadcastScore` message that
takes `Encode.Value` as an argument:

```elm
type Msg
    = BroadcastScore Encode.Value
    | CountdownTimer Time.Posix
    | GameLoop Float
    | KeyDown String
    | NoOp
    | SetNewItemPositionX Int
```

Then, we'll add to our `update` function with the following:

```elm
update : Msg -> Model -> ( Model, Cmd Msg )
update msg model =
    case msg of
        BroadcastScore value ->
            ( model, broadcastScore value )

        -- ...
```

So far so good. We've got a port called `broadcastScore` configured to send
our score as a JSON value out to JavaScript. And now that we have
`BroadcastScore` in our `update` function, we have a way to trigger the port
to send out the data.

## Triggering the Port to Send Data

Let's create a button at the bottom of our view that we can use to trigger
`BroadcastScore` and send the `playerScore` data from our model.

We'll call the button `viewBroadcastScoreButton` with text that says
`"Broadcast Score Over Socket"`. We're using a `let` expression to give a
clearer indication of how the data flows through. We start with the current
score stored in `model.playerScore`, which gets encoded as an integer with
`Encode.int`. Then, we send that to `BroadcastScore` and trigger it when we
click the button.

```elm
viewBroadcastScoreButton : Model -> Html Msg
viewBroadcastScoreButton model =
    let
        broadcastEvent =
            model.playerScore
                |> Encode.int
                |> BroadcastScore
                |> Html.Events.onClick
    in
    button
        [ broadcastEvent
        , Html.Attributes.class "button"
        ]
        [ text "Broadcast Score Over Socket" ]
```

Don't forget that we'll need to add this to our main `view` function to see it
appear when we load our application:

```elm
view : Model -> Html Msg
view model =
    div [ class "container" ]
        [ viewGame model
        , viewBroadcastScoreButton model
        ]
```

## Seeing the Results with JavaScript

We might be tempted to load the application and try things out now, but we
still need to set up the JavaScript side before we see the data that we're
sending from Elm.

Let's open our `assets/js/app.js` file and use JavaScript to handle the data
we're sending from the Elm side.

At the bottom of the file, we're initializing our `Platformer` game with the
following code:

```javascript
if (platformer) {
  Elm.Games.Platformer.init({ node: platformer });
}
```

Let's update this code so we can assign our application to a variable called
`app` and then use `app.ports.broadcastScore.subscribe()` to subscribe to the
data coming from our Elm application.

```javascript
if (platformer) {
  let app = Elm.Games.Platformer.init({ node: platformer });

  app.ports.broadcastScore.subscribe(function (scoreData) {
    console.log(`Broadcasting ${scoreData} score data from Elm using the broadcastScore port.`);
    // Later, we'll push the score data to the Phoenix channel
  });
}
```

If we start our server and load the game we're now at a point where we can test
things out. You can start a game and collect a couple coins to get a value for
the `playerScore` field. Then, we can click the "Broadcast Score Over Socket"
button and see the `console.log()` statement in the DevTools console. In this
screenshot, the player collected two coins and then triggered the
`broadcastScore` port with the button, which produces the console output.

![Working Elm Port with Console Output](images/working_elm_port.png)

This is great news because it means we now have the ability to send data from
our Elm application outside to JavaScript. We're admittedly not doing anything
with the data yet, but we're on the right track since it's available outside of
our Elm application.

## Initializing the Socket and Joining the Channel

We've made it a long way so far in this chapter. We created our `ScoreChannel`,
which will listen for a `"broadcast_score"` message. And we have our Elm
application sending out score data to JavaScript with the `broadcastScore`
port.

Next, we'll need to initialize the Phoenix socket in our `assets/js/app.js`
file and join the channel (as a side note, there's a file called
`assets/js/socket.js` that can be used to set up the socket connections too.
But we're opting to keep everything in the `app.js` file in this book to make
things easier).

Let's open the `app.js` file and add the following code above the Elm section
to import the Phoenix socket library and start the socket connection.

```javascript
// Phoenix Socket
import { Socket } from "phoenix"

let socket = new Socket("/socket", {})

socket.connect()

// Elm
// ...
```

And now that we're connected to the socket, we can add the code to join the
`score` channel for our `platformer` game. Note that some code has been trimmed
below to highlight the changes. We're successfully connecting to the socket,
and then we're only going to join the channel for the `"score:platformer"`
topic when we're on the page that has the `div` element with our `platformer`
game.

```javascript
// Phoenix Socket
import { Socket } from "phoenix"

let socket = new Socket("/socket", {})

socket.connect()

// Elm
// ...

if (platformer) {
  let app = Elm.Games.Platformer.init({ node: platformer });

  let channel = socket.channel("score:platformer", {})

  channel.join()
    .receive("ok", resp => { console.log("Joined successfully", resp) })
    .receive("error", resp => { console.log("Unable to join", resp) })

  app.ports.broadcastScore.subscribe(function (scoreData) {
    console.log(`Broadcasting ${scoreData} score data from Elm using the broadcastScore port.`);
    // Later, we'll push the score data to the Phoenix channel
  });
}
```

We now have a working socket connection and ability to join the score channel
we created. And when we load the page in the browser we should see the console
output about successfully joining the channel.

![Working Socket Connection and Channel Join](images/working_socket_connection.png)

## Pushing Data Over the Socket

We have everything we need at this point to send data from Elm to JavaScript to
our Phoenix channel. The port is taking the `playerScore` data from the Elm
game and making it available in the `broadcastScore` port in our JavaScript
code. Now, we can use `channel.push()` to send data from JavaScript to the
Phoenix channel.

Let's update the `broadcastScore` port in our `app.js` file to push our
`scoreData` over the channel to the `"broadcast_score"` message that our
channel is listening for.

```javascript
app.ports.broadcastScore.subscribe(function (scoreData) {
  console.log(`Broadcasting ${scoreData} score data from Elm using the broadcastScore port.`);
  channel.push("broadcast_score", { player_score: scoreData });
});
```

We've successfully managed to send data from Elm to JavaScript to Phoenix at
this point. To test it out, you can start the Phoenix server with
`mix phx.server` and load the game in the browser. Collect a few coins and
trigger the broadcast by clicking the button. Not only will we see the messages
in the DevTools console to show that we successfully joined the channel and
sent the score data over the socket, but we can also look at the output for our
Phoenix server and see the results.

```shell
[info] CONNECT PlatformWeb.UserSocket
  Transport: :websocket
  Connect Info: %{}
  Parameters: %{"vsn" => "2.0.0"}
[info] Replied PlatformWeb.UserSocket :ok
[info] JOIN "score:platformer" to PlatformWeb.ScoreChannel
  Transport:  :websocket
  Serializer: Phoenix.Socket.V2.JSONSerializer
  Parameters: %{}
[info] Replied score:platformer :ok
[debug] INCOMING "broadcast_score" on "score:platformer" to PlatformWeb.ScoreChannel
  Parameters: %{"player_score" => 200}
```

We're successfully connecting to the socket and joining the
`"score:platformer"` topic for the `ScoreChannel`. And then we're handling the
`"broadcast_score"` message we triggered with the click of a button to send the
`player_score` data.

We moved quickly through some of these concepts, so feel free to check out the
Phoenix documentation on
[getting started with channels](https://hexdocs.pm/phoenix/channels.html).
Although the examples don't contain any Elm code, they provide a great high
level overview of how channels work and show the intersection of how Phoenix
and JavaScript can work well together.

## Summary

We made it a _long_ way in this chapter. We managed to take care of most of the
hard work, and we're successfully sending data from the Elm front-end to the
Phoenix back-end. We created a new Phoenix channel, set up a port from Elm to
JavaScript, and sent the score data from Elm to JavaScript to Phoenix.

Although we're sending data out of our Elm application, we haven't set up
another port to receive data back in so we can display the scores on the page.
And we don't have a way to differentiate between which players are connected to
the socket and playing a game yet. We'd also like to give users the option to
save their scores to the database and persist the gameplay data. We'll take a
look at these topics in the next couple chapters.
