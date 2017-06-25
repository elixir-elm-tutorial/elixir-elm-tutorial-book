# Displaying Game Data

...

## Scoring with Item Collection

Let's add a score indicator to the view for our game, and we'll start by adding
a `playerScore` field to our model. We'll set it to an initial value of `0`.

```elm
type alias Model =
    { playerScore : Int
    , characterPositionX : Int
    , characterPositionY : Int
    , itemPositionX : Int
    , itemPositionY : Int
    }


initialModel : Model
initialModel =
    { playerScore = 0
    , characterPositionX = 50
    , characterPositionY = 300
    , itemPositionX = 500
    , itemPositionY = 300
    }
```

We can start by adding a new function to our view, and then we'll add the
functionality for tracking our player's score based on the number of coins they
pick up. We'll use an SVG `text` element (which is called `Svg.text_` to
differentiate between the actual text we're using with `Svg.text`) to display
the score at the top left of the game window. So we can start by creating a new
`viewGameScore` function beneath the `viewGameWindow` function:

```elm
viewGameScore : Model -> Svg Msg
viewGameScore model =
    Svg.text_
        [ x "490"
        , y "20"
        , fontFamily "Impact"
        , fontSize "16"
        ]
        [ Svg.text ("SCORE: " ++ model.playerScore) ]
```

And we'll need to add this to the bottom of our `viewGame` function so we can
see it rendered:

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