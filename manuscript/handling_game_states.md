# Handling Game States

Since this isn't book about game programming, we won't have time to delve too
deeply into the topic of game design. But we still want our minigames to be
fun, and one way to do this is to be thoughtful about gameplay.

## Game State

Let's begin by thinking about how we want our game to start, and also how we
want it to end. It looks like our game is going to be a simple race against the
clock to collect items. We don't have some amazing plot to add yet, because we
really want to focus on the core gameplay being fun first.

The starting state should be pretty straightforward. We just want a simple text
screen that appears with some basic instructions, and perhaps an indicator of
which key to press to start the game.

We'll also want to add states for when the game is currently being played,
a success state for when the player wins, and a game over screen.

## Union Types

This is a great opportunity to talk about union types in Elm. Types can be
a really helpful way to think about something that has a limited set of
possible states. In our case, we want to create a `GameState` type that handles
all the scenarios we mentioned above. You can add this type right above the
`Model` type alias:

```elm
type GameState = StartScreen | Playing | Success | GameOver
```

I like to think of union types in this way on a single line. We're creating a
`GameState` type that has four possibilities, and it's easy to reason about.
Once the code gets formatted it should look like this instead:

```elm
type GameState
    = StartScreen
    | Playing
    | Success
    | GameOver
```

And we can use this new type in our model to indicate the game's current state.
We'll initialize the state to the game's `StartScreen`, which we'll create
soon.

```elm
type GameState
    = StartScreen
    | Playing
    | Success
    | GameOver


type alias Model =
    { gameState : GameState
    , characterPositionX : Int
    , characterPositionY : Int
    , itemPositionX : Int
    , itemPositionY : Int
    , itemsCollected : Int
    , playerScore : Int
    , timeRemaining : Int
    }


initialModel : Model
initialModel =
    { gameState = StartScreen
    , characterPositionX = 50
    , characterPositionY = 300
    , itemPositionX = 500
    , itemPositionY = 300
    , itemsCollected = 0
    , playerScore = 0
    , timeRemaining = 10
    }
```

## Adjusting the View

Now we want to update our `viewGame` function to account for the different game
states that we've created. But before we make changes to that function, let's
create a new function called `viewGameStates` below it that will handle our
cases. The idea is that we'll take in the `model` as an argument, and then
we'll display a list of SVG content based on the current state of the game.

```elm
viewGameState : Model -> List (Svg Msg)
viewGameState model =
    case model.gameState of
        StartScreen ->
            []

        Playing ->
            []

        Success ->
            []

        GameOver ->
            []
```

We started with empty lists for each of these cases, but let's go ahead and
fill in the `Playing` state since we already had the content for that one.
While users are actively playing the game, we'll want them to be able to see
all the relevant view functions we created for the game:

```elm
viewGameState : Model -> List (Svg Msg)
viewGameState model =
    case model.gameState of
        StartScreen ->
            []

        Playing ->
            [ viewGameWindow
            , viewGameSky
            , viewGameGround
            , viewCharacter model
            , viewItem model
            , viewGameScore model
            , viewItemsCollected model
            , viewGameTime model
            ]

        Success ->
            []

        GameOver ->
            []
```

Now we can update our `viewGame` function to reference this new function that
we just created. This might seem a bit confusing at first, but the idea is that
we're keeping the main `svg` element for our game, and then we're using the
`viewGameState` function to populate it with whatever elements are relevant to
the current state. Here's the new `viewGame` function:

```elm
viewGame : Model -> Svg Msg
viewGame model =
    svg [ version "1.1", width "600", height "400" ]
        (viewGameState model)
```

## Starting Screen

For the start screen, we'll still show our basic game window with the sky, the
ground, the character, and the item. And we also want to display some text to
the player to get an idea for what to accomplish. Let's add a `viewStartScreen`
function that will display some introductory text:

```elm
viewStartScreenText : Svg Msg
viewStartScreenText =
    Svg.svg []
        [ viewGameText 140 160 "Collect ten coins in ten seconds!"
        , viewGameText 140 180 "Press the SPACE BAR key to start."
        ]
```

Now we can add this at the bottom of the `StartScreen` state in our
`viewGameState` function.

```elm
viewGameState : Model -> List (Svg Msg)
viewGameState model =
    case model.gameState of
        StartScreen ->
            [ viewGameWindow
            , viewGameSky
            , viewGameGround
            , viewCharacter model
            , viewItem model
            , viewStartScreenText
            ]

        Playing ->
            [ viewGameWindow
            , viewGameSky
            , viewGameGround
            , viewCharacter model
            , viewItem model
            , viewGameScore model
            , viewItemsCollected model
            , viewGameTime model
            ]

        Success ->
            []

        GameOver ->
            []
```

The structure should be getting a little clearer now. We're basically going to
keep using the `viewGameState` function as our way to selectively show
different components of our game. In this case, we want players to start with
some brief instructions, and they'll be able to see the character and the item
they're going after.

![Start Screen](images/handling_game_states/start_screen.png)

The instructions mention using the space bar key to start the game, so let's
add that feature now. It will also be useful as a way to reset to default
values when we need to.

We already have a `KeyDown` message that we've been using to track keyboard
input, so we can use that to look for when the user presses the space bar key,
which has the key code of `32`.

Update the contents of the `KeyDown` action with the following:

```elm
KeyDown keyCode ->
    case keyCode of
        32 ->
            ( { model | gameState = Playing }, Cmd.none )

        37 ->
            ( { model | characterPositionX = model.characterPositionX - 15 }, Cmd.none )

        39 ->
            ( { model | characterPositionX = model.characterPositionX + 15 }, Cmd.none )

        _ ->
            ( model, Cmd.none )
```

Rather than just setting the `gameState` to `Playing`, we'll take this
opportunity to reset a few other values to make sure the player is starting
with the correct time and score. And we'll make sure players don't accidentally
reset the game by pressing the space bar in the middle of playing by wrapping
it in a conditional.

```elm
KeyDown keyCode ->
    case keyCode of
        32 ->
            if model.gameState == StartScreen then
                ( { model
                    | gameState = Playing
                    , playerScore = 0
                    , itemsCollected = 0
                    , timeRemaining = 10
                    }
                , Cmd.none
                )
            else
                ( model, Cmd.none )

        37 ->
            ( { model | characterPositionX = model.characterPositionX - 15 }, Cmd.none )

        39 ->
            ( { model | characterPositionX = model.characterPositionX + 15 }, Cmd.none )

        _ ->
            ( model, Cmd.none )
```

## Success State

If a player has managed to collect ten coins before the time has expired, then
we can consider the level completed and display a success message.

Let's create a new function called `viewSuccessScreenText` with the following
content:

```elm
viewSuccessScreenText : Svg Msg
viewSuccessScreenText =
    Svg.svg []
        [ viewGameText 120 180 "Success!"
        ]
```

If we had time, it would be fun to add some SVG fireworks or animations here
for the game's success state. But for now we'll just add some text like we did
for the starting screen.

Let's update the `viewGameState` function with the following:

```elm
viewGameState : Model -> List (Svg Msg)
viewGameState model =
    case model.gameState of
        StartScreen ->
            [ viewGameWindow
            , viewGameSky
            , viewGameGround
            , viewCharacter model
            , viewItem model
            , viewStartScreenText
            ]

        Playing ->
            [ viewGameWindow
            , viewGameSky
            , viewGameGround
            , viewCharacter model
            , viewItem model
            , viewGameScore model
            , viewItemsCollected model
            , viewGameTime model
            ]

        Success ->
            [ viewGameWindow
            , viewGameSky
            , viewGameGround
            , viewCharacter model
            , viewItem model
            , viewSuccessScreenText
            ]

        GameOver ->
            []
```

In order to trigger the success screen, we'll need to create another condition
in our `TimeUpdate` for when the user's score arrives at a value of `10`.
Inside the `update` function, adapt the code with the following:

```elm
TimeUpdate time ->
    if characterFoundItem model then
        ( { model
            | itemsCollected = model.itemsCollected + 1
            , playerScore = model.playerScore + 100
            }
        , Random.generate SetNewItemPositionX (Random.int 50 500)
        )
    else if model.itemsCollected >= 10 then
        ( { model | gameState = Success }, Cmd.none )
    else
        ( model, Cmd.none )
```

We also want our players to be able to restart the game without having to
refresh the page in the browser. Let's update the `viewSuccessScreenText`
function we just created to include some more text about restarting the game:

```elm
viewSuccessScreenText : Svg Msg
viewSuccessScreenText =
    Svg.svg []
        [ viewGameText 260 160 "Success!"
        , viewGameText 140 180 "Press the SPACE BAR key to restart."
        ]
```

And now we can update the space bar case in our `update` function` so that
users can restart the game from the `Success` state.

```elm
KeyDown keyCode ->
    case keyCode of
        32 ->
            case model.gameState of
                StartScreen ->
                    ( { model
                        | gameState = Playing
                        , playerScore = 0
                        , itemsCollected = 0
                        , timeRemaining = 10
                        }
                    , Cmd.none
                    )

                Success ->
                    ( { model
                        | gameState = Playing
                        , playerScore = 0
                        , itemsCollected = 0
                        , timeRemaining = 10
                        }
                    , Cmd.none
                    )

                _ ->
                    ( model, Cmd.none )
```

At this point, we should have working states for the start screen, the active
playing state, and the success screen.

![Success Screen](images/handling_game_states/success_screen.png)

## Game Over State

We still haven't figured out what to do when the timer reaches zero and the
player hasn't collected enough coins. In this case, we'll take a similar
approach to the one we used for the success state in that we want to show some
game over text along with an option for the player to restart the game.

We'll start with a simple `viewGameOverScreenText` function for the text we
want to display:

```elm
viewGameOverScreenText : Svg Msg
viewGameOverScreenText =
    Svg.svg []
        [ viewGameText 260 160 "Game Over"
        , viewGameText 140 180 "Press the SPACE BAR key to restart."
        ]
```

Now we can finish up our `viewGameState` function. We're only updating the
`GameOver` case, but the full code is posted here for a review of what we're
selectively displaying for each state.

```elm
viewGameState : Model -> List (Svg Msg)
viewGameState model =
    case model.gameState of
        StartScreen ->
            [ viewGameWindow
            , viewGameSky
            , viewGameGround
            , viewCharacter model
            , viewItem model
            , viewStartScreenText
            ]

        Playing ->
            [ viewGameWindow
            , viewGameSky
            , viewGameGround
            , viewCharacter model
            , viewItem model
            , viewGameScore model
            , viewItemsCollected model
            , viewGameTime model
            ]

        Success ->
            [ viewGameWindow
            , viewGameSky
            , viewGameGround
            , viewCharacter model
            , viewItem model
            , viewSuccessScreenText
            ]

        GameOver ->
            [ viewGameWindow
            , viewGameSky
            , viewGameGround
            , viewCharacter model
            , viewItem model
            , viewGameOverScreenText
            ]
```

We'll also need to add another condition for when the timer reaches `0` and the
player has collected less than `10` coins. Let's update the `TimeUpdate`
message to set the `gameState` to `GameOver` when those conditions arise during
gameplay:

```elm
TimeUpdate time ->
    if characterFoundItem model then
        ( { model
            | itemsCollected = model.itemsCollected + 1
            , playerScore = model.playerScore + 100
            }
        , Random.generate SetNewItemPositionX (Random.int 50 500)
        )
    else if model.itemsCollected >= 10 then
        ( { model | gameState = Success }, Cmd.none )
    else if model.itemsCollected < 10 && model.timeRemaining == 0 then
        ( { model | gameState = GameOver }, Cmd.none )
    else
        ( model, Cmd.none )
```

Lastly, we want users to be able to restart gameplay from the `GameOver` state.
We can also go ahead and refactor our cases quite a bit here since a lot of
them are intended to reset the game state. Update the `KeyDown` with the
following:

```elm
KeyDown keyCode ->
    let
        resetGameState =
            ( { model
                | gameState = Playing
                , playerScore = 0
                , itemsCollected = 0
                , timeRemaining = 10
                }
            , Cmd.none
            )
    in
        case keyCode of
            32 ->
                case model.gameState of
                    StartScreen ->
                        resetGameState

                    Success ->
                        resetGameState

                    Playing ->
                        ( model, Cmd.none )

                    GameOver ->
                        resetGameState

            37 ->
                ( { model | characterPositionX = model.characterPositionX - 15 }, Cmd.none )

            39 ->
                ( { model | characterPositionX = model.characterPositionX + 15 }, Cmd.none )

            _ ->
                ( model, Cmd.none )
```

## Summary

So far so good! We have working game states now so that players can see a start
screen, play the game, view a success state, and also restart in the game over
state. This is a great start, but let's keep working towards improving our
gameplay and injecting some fun in the next chapter.
