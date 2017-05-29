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

From the command-line, let's switch to the `lib/platform/web/elm` folder and
run the following command:

```shell
$ elm-package install elm-lang/keyboard
```

After agreeing to install the package by entering the `Y` key, here's the
output we should see:

```shell
$ elm elm-package install elm-lang/keyboard
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
`Game.elm` file. We'll need to import `KeyCode`s along with the `presses`
function. The way it works is that each key on your keyboard is represented by
an integer. The Elm core library comes with functions called `fromCode` and
`toCode` to convert back and forth between keyboard keys and their related
integer representations.

As an example, we're going to want our character to move right when we press
the right arrow key on the keyboard. That key is represented by the integer
`39` (you can use http://keycode.info to type on your keyboard and see the
related integer value, but these values are easy to look up so we don't need
to memorize them or anything). In our application, we'll be able to determine
that users are pressing the right arrow key, and adjust our character's
position accordingly.

Let's update the top of our `Game.elm` file with the following:

```elm
module Game exposing (..)

import Html exposing (Html, div)
import Keyboard exposing (KeyCode, presses)
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
    Sub.batch
        [ presses KeyPress ]
```

One thing to keep in mind is that this code won't work until we add `KeyPress`
to our `update` function. The `Sub.batch` function allows us to batch together
different subscriptions, so we could also subscribe to mouse input if we needed
to. For now, all we need to know is that we're using the Elm Architecture to
subscribe to keyboard input via the `presses` function, and we're going to
handle these presses with the `KeyPress` message in the `update` function.

As an initial way to get keyboard input working, we're going to set things up
so that any keypress will move the character slightly to the right on the
screen (towards the coin item). To accomplish this, we'll start by creating our
new `KeyPress` update message, that takes a `KeyCode` as an argument:

```elm
type Msg
    = NoOp
    | KeyPress KeyCode
```

We have two update actions, the first is to perform no operation with `NoOp`,
and the second will perform an update to the model based on `KeyPress` actions
from the user.

Let's finish getting things working again with a big change to our `update`
function:

```elm
update : Msg -> Model -> ( Model, Cmd Msg )
update msg model =
    case msg of
        NoOp ->
            ( model, Cmd.none )

        KeyPress keyCode ->
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
`{ model | characterPositionX = model.characterPositionX + 15 }`. We set up our
model as a record with several fields in it. With this syntax, we can change
the value of the `characterPositionX` field in the model. It takes some getting
used to, but for now all we need to know is that we're setting a new value to
`characterPositionX` every time we press a key on the keyboard.

In the `initialModel`, we set our `characterPositionX` value to `50`. Now with
every key press we're increasing that value by `15`. So if you press any key
four times, the character will move to the right by 60 pixels.

[NOTE: Review why some keys aren't working.]

## Creating a Key Module

Because we'd want to use keyboard input in many games, we can create a new
`Key` module that's reusable.

In our `lib/web/elm` folder, let's create a new file called `Key.elm` that will
contain our module.

```elm
module Key exposing (..)


type Key
    = SpaceBar
    | ArrowLeft
    | ArrowUp
    | ArrowRight
    | ArrowDown
    | Unknown


fromCode : Int -> Key
fromCode keyCode =
    case keyCode of
        32 ->
            SpaceBar

        37 ->
            ArrowLeft

        38 ->
            ArrowUp

        39 ->
            ArrowRight

        40 ->
            ArrowDown

        _ ->
            Unknown
```

...