# Displaying Game Data

Let's start thinking about how we want to display game data to our players.
We'll add text to our game window to indicate the player's score, the number of
items collected, and add the concept of time to our game.

## Scoring with Item Collection

Our player's score and the number of items that the character collects are
values that will change, and any time we're dealing with changing values it's
a good sign that we want to track those values in our model.

Let's get started by adding a `playerScore` to our model:

```elm
type alias Model =
    { characterPositionX : Int
    , characterPositionY : Int
    , itemPositionX : Int
    , itemPositionY : Int
    , playerScore : Int
    }


initialModel : Model
initialModel =
    { characterPositionX = 50
    , characterPositionY = 300
    , itemPositionX = 500
    , itemPositionY = 300
    , playerScore = 0
    }
```

We start with an initial value of `0`, and the player will increase this value
as she collects new items. In fact, now would be a good time to add another
field to track the number of items the character has collected. Let's go ahead
and add an `itemsCollected` field too:

```elm
type alias Model =
    { characterPositionX : Int
    , characterPositionY : Int
    , itemPositionX : Int
    , itemPositionY : Int
    , itemsCollected : Int
    , playerScore : Int
    }


initialModel : Model
initialModel =
    { characterPositionX = 50
    , characterPositionY = 300
    , itemPositionX = 500
    , itemPositionY = 300
    , itemsCollected = 0
    , playerScore = 0
    }
```

## Rendering Text Data

Now that we have some initial values for our player's score and the number of
items collected, we can add a score indicator to the view for our game.

First, let's start by adding a view helper function that we can use to set some
standards for how we want our text to look. We'll create a function called
`viewGameText` that will allow us to render some text in the game window, and
we can use arguments to indicate the position and the string of text that we
want to display:

```elm
viewGameText : Int -> Int -> String -> Svg Msg
viewGameText positionX positionY str =
    Svg.text_
        [ x (toString positionX)
        , y (toString positionY)
        , fontFamily "Courier"
        , fontWeight "bold"
        , fontSize "16"
        ]
        [ Svg.text str ]
```

The first thing to note about this function is that we're using `Svg.text_`,
which is the SVG `<text>` element (whereas `Svg.text` is to create the actual
string of text we're going to display). Then we set some simple font attributes
so our text will look nice. This function will allow us to move text around
easily without having to duplicate all this code over and over again as we add
new text indicators.

The first thing we want to display is our player's score in the upper left of
the game window. To accomplish this, we'll create a new `viewGameScore`
function that takes in the current `model` and return some SVG with the text
we want to display.

There might be a handful of unfamiliar things that we're not used to seeing
here, but what we're accomplishing is actually very straightforward. We want
to start with the `playerScore` integer value from the model (which starts out
with a value of `0`). We use a `let` expression to convert this into a string
with `toString` since that's what we'll need in order to display it as text in
our game window. Then, we use the `String.padLeft` function from the `String`
module to display some leading zero characters in our score. This isn't
strictly necessary, but helps make our score display look a little nicer.
Lastly, we take the `currentScore` value from our `let` expression, and we
put that inside a group of `viewGameText` functions that we'll use to display
everything. As for how the position values were determined, it was mainly just
tinkering with those numbers after adding this to the page and experimenting
with what looks good. Here's the `viewGameScore` function:

```elm
viewGameScore : Model -> Svg Msg
viewGameScore model =
    let
        currentScore =
            model.playerScore
                |> toString
                |> String.padLeft 5 '0'
    in
        Svg.svg []
            [ viewGameText 25 25 "SCORE"
            , viewGameText 25 40 currentScore
            ]
```

Now we can just call this new function at the bottom of the `viewGame` function
and we should be able to see the score rendered in the browser:

```elm
viewGame : Model -> Svg Msg
viewGame model =
    svg [ version "1.1", width "600", height "400" ]
        [ viewGameWindow
        , viewGameSky
        , viewGameGround
        , viewCharacter model
        , viewItem model
        , viewGameScore model
        ]
```

![Displaying Score Text](images/displaying_game_data/displaying_score_text.png)

## Displaying Items Collected

We'll take a similar approach to show the number of coins the character has
collected.