# Movement

When it comes to moving our character around on the screen, we've been kind of
cheating. We've just been manually altering the position when the left and
right arrow keys are pressed, and shifting the character's position. What we'd
really like is to tinker with our character's velocity, which is the speed in a
certain direction. This is going to require quite a few changes, but it will be
worth it in terms of moving towards a game that's more fun to play.

## Velocity

Let's start by updating our model and adding a new field for the character's
velocity as a `Float`. In fact, we'll also need to update the other fields for
our character's position to be the same type.

```elm
type alias Model =
    { gameState : GameState
    , characterPositionX : Float
    , characterPositionY : Float
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
    , characterPositionX = 50.0
    , characterPositionY = 300.0
    , characterVelocity = 0.0
    , itemPositionX = 500
    , itemPositionY = 300
    , itemsCollected = 0
    , playerScore = 0
    , timeRemaining = 10
    }
```

This will create some issues in our program, because we were previously using
`Int` types. Thankfully, Elm guides us gently to making the necessary changes.
Let's update our `characterFoundItem` function to make the necessary
conversions between `Float` and `Int` types so that our character can still
discover and collect items.

```elm
characterFoundItem : Model -> Bool
characterFoundItem model =
    let
        approximateItemLowerBound =
            toFloat model.itemPositionX - 35

        approximateItemUpperBound =
            toFloat model.itemPositionX

        currentCharacterPosition =
            model.characterPositionX
    in
        currentCharacterPosition
            >= approximateItemLowerBound
            && currentCharacterPosition
            <= approximateItemUpperBound
```

We're changing our approach here slightly. Instead of creating a list of
integers to determine whether our character has stumbled on an item, we're
determining whether our character's position is greater than the lower bound
of the item and less than the approximate upper bound. Still not a perfect
solution, but it works to get our game functioning properly again.

## Adjusting Arrow Keys

Now we want to adjust our `characterVelocity` with our left and right arrow
keys instead of changing the position. Let's update the cases for the left and
right arrow keys (`37` and `39`) in our `KeyDown` message:

```elm
37 ->
    ( { model | characterVelocity = model.characterVelocity - 0.25 }, Cmd.none )

39 ->
    ( { model | characterVelocity = model.characterVelocity + 0.25 }, Cmd.none )
```

Keep in mind that this will break our character's previous ability to move
around the screen, so we'll take a look at fixing that now.

Let's go ahead and add a new `MoveCharacter` message to the bottom of our
update section. We'll already have access to the fields from the model in this
context, but we'll need these changes to happen over time, so we want to pass
an argument of type `Time` too.

```elm
type Msg
    = NoOp
    | KeyDown KeyCode
    | TimeUpdate Time
    | CountdownTimer Time
    | SetNewItemPositionX Int
    | MoveCharacter Time
```

Now we can add a new case at the bottom of our `update` function to account
for character movement. We're going to apply the changes in the character's
velocity to the character's position over time. This will also allow us to
alter the character's speed if we want to implement a different speed for the
character walking and running.

```elm
MoveCharacter time ->
    ( { model | characterPositionX = model.characterPositionX + model.characterVelocity * time }, Cmd.none )
```

Lastly, we can add `MoveCharacter` to our `subscriptions` function so that
we're animating our character's movement:

```elm
subscriptions : Model -> Sub Msg
subscriptions model =
    Sub.batch
        [ downs KeyDown
        , diffs TimeUpdate
        , diffs MoveCharacter
        , every second CountdownTimer
        ]
```

Go ahead and tinker around with the character's movement in the browser. It
looks like we managed to successfully add velocity to our game, but there were
also some unintended consequences. On the one hand, we can now move the
character back and forth, and we can even vary the speed wildly by pressing the
same arrow twice. But we also have to press the key in the opposite direction
in order to stop movement.

...

## TODO

- Character velocity and direction
- Character running and jumping ability
- Switch from random locations to specific patterns
- Player should learn new skill at each level
- Add different levels and final success state
