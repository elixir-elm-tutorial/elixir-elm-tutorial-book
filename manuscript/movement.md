# Movement

When it comes to moving our character around on the screen, we've been kind of
cheating. We've just been manually altering the position when the left and
right arrow keys are pressed, and shifting the character's position. What we'd
really like is to tinker with our character's velocity, which is the speed in a
certain direction. This is going to require quite a few changes, but it will be
worth it in terms of moving towards a game that's more fun to play.

## Velocity

Let's start by updating our model and adding a new field for the character's
`velocity` as a `Float`.

In fact, we'll need to update all the fields for our character and item
positions to all be `Float` types.

```elm
type alias Model =
    { gameState : GameState
    , characterPositionX : Int
    , characterPositionY : Int
    , characterVelocity : Float
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
    , characterVelocity = 0
    , itemPositionX = 500
    , itemPositionY = 300
    , itemsCollected = 0
    , playerScore = 0
    , timeRemaining = 10
    }
```

Inside our `update` function, we want to adjust what happens when we press the
left and right arrow keys. Instead of updating our character's position, we'll
alter the `velocity`. Keep in mind that this will break our game for the
moment, because we're only adjusting the `characterVelocity` and we're going to
move the position adjustment elsewhere. Update the code inside the `KeyDown`
message so that the cases for `37` and `39` look like the following:

```elm
37 ->
    ( { model | characterVelocity = model.characterVelocity - 1.0 }
    , Cmd.none
    )

39 ->
    ( { model | characterVelocity = model.characterVelocity + 1.0 }
    , Cmd.none
    )
```

## TODO

- Character velocity and direction
- Character running and jumping ability
- Switch from random locations to specific patterns
- Player should learn new skill at each level
- Add different levels and final success state
