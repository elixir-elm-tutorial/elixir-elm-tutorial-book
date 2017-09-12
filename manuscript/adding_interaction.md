# Adding Interaction

In the last chapter, we managed to set up our first game. Now, we can start
adding interactivity to our game to make it come to life. We're going to
allow our players to interact with our character via keyboard input, and we'll
learn about Elm subscriptions along the way.

## Subscriptions

When working with the Elm Architecture, subscriptions allow us to work with
streams of data and subscribe to a sequence of events. Keyboard and mouse input
from users are a great examples of how this works. For instance, we can
"subscribe" to the user's mouse position, and it will allow to track the
mouse location as it changes over time. Don't worry if it sounds a little
confusing, we'll take a look at how we can subscribe to keyboard input now.

## Importing the Keyboard Package

In order to work with keyboard input, we'll need to start by importing the Elm
`Keyboard` package.

From the command line, let's switch to the `assets` folder and
run the following command:

```shell
$ elm-package install elm-lang/keyboard
```

After agreeing to install the package by entering the `Y` key, here's the
output we should see:

```shell
$ elm-package install elm-lang/keyboard
To install elm-lang/keyboard I would like to add the following
dependency to elm-package.json:

    "elm-lang/keyboard": "1.0.1 <= v < 2.0.0"

May I add that to elm-package.json for you? [Y/n] Y

Some new packages are needed. Here is the upgrade plan.

  Install:
    elm-lang/dom 1.1.1
    elm-lang/keyboard 1.0.1

Do you approve of this plan? [Y/n] Y
Starting downloads...

  ● elm-lang/keyboard 1.0.1
  ● elm-lang/dom 1.1.1

Packages configured successfully!
```

Now that we have the package installed, let's import it at the top of our
`Game.elm` file. We'll need to import `KeyCode`s along with the `downs`
function. The way it works is that each key on your keyboard is represented by
an integer. The Elm core library comes with functions called `fromCode` and
`toCode` to convert back and forth between keyboard keys and their related
integer representations.

As an example, we're going to want our character to move right when we press
the right arrow key on the keyboard. That key is represented by the integer
`39` (you can use http://keycode.info to type on your keyboard and see the
related integer value, but these values are easy to look up so we don't need
to memorize them). In our application, we'll be able to determine that users
are pressing the right arrow key, and adjust our character's position
accordingly.

Let's update the top of our `Game.elm` file with the following:

```elm
module Game exposing (..)

import Html exposing (Html, div)
import Keyboard exposing (KeyCode, downs)
import Svg exposing (..)
import Svg.Attributes exposing (..)
```

## Tracking Key Presses

In order to track key presses, we'll need to make two changes:

- subscribe to key presses in our `subscriptions` function
- handle the key presses in our `update` function

Let's start with the `subscriptions` function. Instead of `Sub.none`, let's
subscribe to key presses with the following code:

```elm
subscriptions : Model -> Sub Msg
subscriptions model =
    Sub.batch [ downs KeyDown ]
```

One thing to keep in mind is that this code won't work until we add `KeyDown`
to our `update` function. The `Sub.batch` function allows us to batch together
different subscriptions, so we could also subscribe to mouse input if we needed
to. For now, all we need to know is that we're using the Elm Architecture to
subscribe to keyboard input via the `presses` function, and we're going to
handle these presses with the `KeyDown` message in the `update` function.

As an initial way to get keyboard input working, we're going to set things up
so that any key press will move the character slightly to the right on the
screen (towards the coin item). To accomplish this, we'll start by creating our
new `KeyDown` update message, that takes a `KeyCode` as an argument:

```elm
type Msg
    = NoOp
    | KeyDown KeyCode
```

We have two update actions, the first is to perform no operation with `NoOp`,
and the second will perform an update to the model based on `KeyDown` actions
from the user.

Let's finish getting things working again with a big change to our `update`
function:

```elm
update : Msg -> Model -> ( Model, Cmd Msg )
update msg model =
    case msg of
        NoOp ->
            ( model, Cmd.none )

        KeyDown keyCode ->
            ( { model | characterPositionX = model.characterPositionX + 15 }, Cmd.none )
```

There's a lot going on here, so don't worry if it seems a little overwhelming
at first. The best way to think about the `update` function is that it takes in
the existing model as an argument, applies an update message, and returns the
new updated version of the model. In this case, we start with our character at
the initial starting position, and with each key press we're going to update
that position and see that value change through the model.

The way this works is that we're using Elm's record update syntax, and we can
just focus on this part of the code for now:

```elm
{ model | characterPositionX = model.characterPositionX + 15 }
```

We set up our model as a record with several fields in it. With this syntax, we
can change the value of the `characterPositionX` field in the model. It takes
some getting used to, but for now all we need to know is that we're setting a
new value to `characterPositionX` every time we press a key on the keyboard.

In the `initialModel`, we set our `characterPositionX` value to `50`. Now with
every key press we're increasing that value by `15`. So if you press any key
four times, the character will move to the right by a total of 60 pixels.

## Setting the Correct Keys

It's exciting to see our character moving around the screen, but we want to
be able to change the direction based on the key we're pressing.

In order to accomplish this, let's add a `case` expression inside our `update`
function. We only want our character to move to the right when we press the
right arrow key (which has the key code `39`) on our keyboard. We're already
passing the `keyCode` to our `KeyDown` message, so we can use our `case`
expression to check which key is being pressed and respond with the following:

```elm
update : Msg -> Model -> ( Model, Cmd Msg )
update msg model =
    case msg of
        NoOp ->
            ( model, Cmd.none )

        KeyDown keyCode ->
            case keyCode of
                39 ->
                    ( { model | characterPositionX = model.characterPositionX + 15 }, Cmd.none )

                _ ->
                    ( model, Cmd.none )
```

Our game is basically performing the same way it was before, where the
character can move to the right. But now that action should only be triggered
when we press down on the right arrow key.

In other words, we're adding `15` to the `characterPositionX` value every time
we press the right arrow key on the keyboard. And the `_` part of the `case`
expression allows us to handle all other scenarios (any other key presses). And
we'll make no change to the model when any other key than the right arrow is
being pressed.

One of the reasons that Elm is such a strong language and offers so many
guarantees is that it forces us to account for all possibilities. So when we
define the behavior we want for the right arrow key, it makes sure that we
don't forget all the other keys and wants us to think about what other behavior
we would want when other keys are pressed. In this case, it's fairly
straightforward because we want the character to move right when the right
arrow key is pressed and for no action to happen with any other keys.

As an aside, it's normally considered good practice to explicitly account for
possibilities. So if we were creating a `case` expression that only had a few
possibilities, we'd try to add a separate conditions for each one to handle
them thoughtfully. But in the case of a keyboard we're only hoping to use a
couple of the keys for now, so we're going to default to no action when most of
the keys are pressed.

## Changing Direction

Let's go ahead and add the ability for our character to move in the left
direction as well. This will involve some familiar changes to our `KeyDown`
message, where we're going to add a case for the left arrow key (`37`) and
subtract from the character's horizontal position value:

```elm
KeyDown keyCode ->
    case keyCode of
        37 ->
            ( { model | characterPositionX = model.characterPositionX - 15 }, Cmd.none )

        39 ->
            ( { model | characterPositionX = model.characterPositionX + 15 }, Cmd.none )

        _ ->
            ( model, Cmd.none )
```

So far so good. We now have the ability to move our character to the left and
the right on the screen. It's great that we've already managed to add some
interactivity to our game, but keep in mind that we're taking some shortcuts
for now. As we continue, it will be nice to account for our character's
acceleration instead of just manually pushing the position around using pixel
values. We'll work towards adding more complex behaviors, but for now let's
continue adding some basic game elements.

## Collecting Items

We now have the ability to move our character to the left and right, and we
already added an item that our character should be able to pick up. Currently,
our character can move over to the item, but nothing really happens when we get
there.

What we'd like to do at this point is to be able to move the character to the
item, increment our score, and spawn a new item.

One way that we can start thinking about this is to figure out what we want to
do when the character reaches the item. Let's add a `characterFoundItem`
function below our `update` function that will return a boolean value about
whether or not the character has discovered the item. We'll use the position of
both the character and the item to see if they match, and then we'll return
`True` if the character's position matches the item's position.

```elm
characterFoundItem : Model -> Bool
characterFoundItem model =
    model.characterPositionX == model.itemPositionX
```

This seems like a good idea initially, but it uncovers some limitations of our
current approach. To see this in action, let's temporarily update the
`viewItem` function to account for our new `characterFoundItem` function. If
the character has found the item, we'll simply return an empty `svg` element to
effectively hide the coin image. Otherwise, we'll continue showing the item if
the character has not found the item.

```elm
viewItem : Model -> Svg Msg
viewItem model =
    case characterFoundItem model of
        True ->
            svg [] []

        False ->
            image
                [ xlinkHref "/images/coin.svg"
                , x (toString model.itemPositionX)
                , y (toString model.itemPositionY)
                , width "20"
                , height "20"
                ]
                []
```

If you try this out in the browser, you'll see that it _roughly_ accomplishes
our goal of having the character be able to "find" the item and have it
disappear. But you'll notice there's a small problem where the character needs
to arrive at a particular position to find the item.

![Character Finding Item](images/adding_interaction/item_found.png)

![Character Finding Item](images/adding_interaction/item_found_issue.png)

We could probably look for an ideal fix for this issue, but for now let's keep
in mind that our current goal is to just make the game playable and to track
the player's score. So let's find a workable solution that will involve giving
the item a _range_ instead of an exact position. And we'll use this opportunity
to learn to use a few new functions from the `List` module.

Instead of using the exact `model.itemPositionX` value like we did above, we
want to add a lower and upper bound for where the character should be able to
find the item. We'll use a `let` expression inside our `characterFoundItem`
function to set values for the `approximateItemLowerBound` and
`approximateItemUpperBound`. Then we'll use the `List.range` function to create
a range of numbers where the character can discover the item.

After we create a range of values where our character can find the item, we use
the `List.member` function to determine whether or not the character position
is currently somewhere inside the item's range. Let's update our
`characterFoundItem` function to see how this works:

```elm
characterFoundItem : Model -> Bool
characterFoundItem model =
    let
        approximateItemLowerBound =
            model.itemPositionX - 35

        approximateItemUpperBound =
            model.itemPositionX

        approximateItemRange =
            List.range approximateItemLowerBound approximateItemUpperBound
    in
        List.member model.characterPositionX approximateItemRange
```

This is generally not great programming practice to use a "magic number" value
like `35` here which is just a rough approximation of where the character
position meets the item position. But tinkering with these values in the
browser looks like it's just good enough to keep moving since the character
is able to discover the item at the correct position, and should improve our
gameplay until we can find a better approach.

## Spawning Items

In order to animate our scene, we'll need to start by importing the Elm
`AnimationFrame` package.

From the command line, let's switch to the `assets` folder and
run the following command:

```shell
$ elm-package install elm-lang/animation-frame
```

After agreeing to install the package by entering the `Y` key, here's the
output we should see:

```shell
$ elm-package install elm-lang/animation-frame
To install elm-lang/animation-frame I would like to add the following
dependency to elm-package.json:

    "elm-lang/animation-frame": "1.0.1 <= v < 2.0.0"

May I add that to elm-package.json for you? [Y/n]

Some new packages are needed. Here is the upgrade plan.

  Install:
    elm-lang/animation-frame 1.0.1

Do you approve of this plan? [Y/n]
Starting downloads...

  ● elm-lang/animation-frame 1.0.1

Packages configured successfully!
```

We now have mechanisms for moving our character around the screen, and we
managed to add some initial code for the character to find items. But we don't
just want to hide the item when the character finds it. We'd like to create
new coins in new locations, and this is an opportunity to start making our game
come to life.

Let's get started by importing a handful of new packages that we'll need for
our game at the top of our file. We'll import `AnimationFrame`, `Random`, and
`Time` (note that I tend to sort my imports in alphabetical order):

```elm
module Game exposing (..)

import AnimationFrame exposing (diffs)
import Html exposing (Html, div)
import Keyboard exposing (KeyCode, downs)
import Random
import Svg exposing (..)
import Svg.Attributes exposing (..)
import Time exposing (Time)
```

The `AnimationFrame` library will be helpful for our game, because it enables
us to render smooth animations and subscribe to differences over time. We can
start by updating our `subscriptions` function to use the `diffs` function from
the `AnimationFrame` library, and we'll pass a new message that we'll create
momentarily. Update `subscriptions` with the following:

```elm
subscriptions : Model -> Sub Msg
subscriptions model =
    Sub.batch
        [ downs KeyDown
        , diffs TimeUpdate
        ]
```

This means we'll need to add a new `TimeUpdate` message along with a change to
our `update` function:

```elm
type Msg
    = NoOp
    | KeyDown KeyCode
    | TimeUpdate Time


update : Msg -> Model -> ( Model, Cmd Msg )
update msg model =
    case msg of
        NoOp ->
            ( model, Cmd.none )

        KeyDown keyCode ->
            case keyCode of
                37 ->
                    ( { model | characterPositionX = model.characterPositionX - 15 }, Cmd.none )

                39 ->
                    ( { model | characterPositionX = model.characterPositionX + 15 }, Cmd.none )

                _ ->
                    ( model, Cmd.none )

        TimeUpdate time ->
            ( model, Cmd.none )
```

This may not seem like a big deal, but it is. We now have the ability to change
things over time in our game, so we could have items and enemies moving around
as time moves forward. Let's update our `TimeUpdate` message to perform some
action. We want to change the position of our item when our character finds it,
so let's use our `characterFoundItem` function to make changes to the `model`:

```elm
TimeUpdate time ->
    if characterFoundItem model then
        ( { model | itemPositionX = model.itemPositionX - 100 }, Cmd.none )
    else
        ( model, Cmd.none )
```

And now that we're using our `characterFoundItem` condition in the `update`
function, we can simplify the `viewItem` function we had temporarily changed
before:

```elm
viewItem : Model -> Svg Msg
viewItem model =
    image
        [ xlinkHref "/images/coin.svg"
        , x (toString model.itemPositionX)
        , y (toString model.itemPositionY)
        , width "20"
        , height "20"
        ]
        []
```

This basically allows the player to move the character to the item's location,
and it will give the appearance that a new coin is being "spawned" in a new
location 100 pixels to the left.

## Working with Randomness

Instead of manually moving the coin to the left, let's take a look at the
`Random` library to move it to a random new location on the x-axis.

To accomplish this, we'll start by changing our `TimeUpdate` message, and then
we'll add a new `SetNewItemPositionX` message to change the position of the
item in the model.

First, let's change our `TimeUpdate` message to remove the manual shifting of
the coin, and replace it with a new random number generator:

```elm
TimeUpdate time ->
    if characterFoundItem model then
        ( model, Random.generate SetNewItemPositionX (Random.int 50 500) )
    else
        ( model, Cmd.none )
```

We're using two different functions from the `Random` library here. We'll use
`Random.int`, which takes two integer values and gives us a random number in
between those values. Then we use the `Random.generate` function, which takes
another message (which we'll use to update the model) along with the random
integer we're creating. It's admittedly confusing, but keep in mind that
generating a random number is actually an effect, because a function that
returns different values depending on the inputs is by nature impure. Working
with random numbers this way allows us to have _managed_ effects and make sure
they don't wreak havoc on our application (we'll learn more about this
elsewhere in the book so don't worry too much for now).

Now that we have a new value, we can use it to update the model with the
`SetNewItemPositionX` message. Here are the changes in context with the message
types and full `update` function:

```elm
type Msg
    = NoOp
    | KeyDown KeyCode
    | TimeUpdate Time
    | SetNewItemPositionX Int


update : Msg -> Model -> ( Model, Cmd Msg )
update msg model =
    case msg of
        NoOp ->
            ( model, Cmd.none )

        KeyDown keyCode ->
            case keyCode of
                37 ->
                    ( { model | characterPositionX = model.characterPositionX - 15 }, Cmd.none )

                39 ->
                    ( { model | characterPositionX = model.characterPositionX + 15 }, Cmd.none )

                _ ->
                    ( model, Cmd.none )

        TimeUpdate time ->
            if characterFoundItem model then
                ( model, Random.generate SetNewItemPositionX (Random.int 50 500) )
            else
                ( model, Cmd.none )

        SetNewItemPositionX newPositionX ->
            ( { model | itemPositionX = newPositionX }, Cmd.none )
```

## Summary

These may not be the most fun game mechanics ever, but we've come a _long_ way
towards building our first game. This is a good stopping point to reflect on
what we've accomplished so far.

We learned about Elm subscriptions and handling keyboard input, allowing the
player to adjust the position of the character on the screen. Then, we imported
new libraries to allow our character to "collect" items and spawn new ones in
different locations.

What's missing is the ability to track the number of items we're collecting.
In the next chapter, we'll cover some basics for tracking game data and
rendering it inside our game window.
