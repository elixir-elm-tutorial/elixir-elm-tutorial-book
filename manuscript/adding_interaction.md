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
function. The way it works is that each key on your keyboard is represented
by an integer. And the Elm core library comes with functions called `fromCode`
and `toCode` to convert back and forth between keyboard keys and their
related integer representations.

As an example, we're going to want our character to move right when we press
the right arrow key on the keyboard. That key is represented by the integer
`39`. In our application, we'll be able to determine that users are pressing
the right arrow key, and adjust our character's position accordingly.

Let's update the top of our `Game.elm` file with the following:

```elm
module Game exposing (..)

import Html exposing (Html, div)
import Keyboard exposing (KeyCode, presses)
import Svg exposing (..)
import Svg.Attributes exposing (..)
```

## Tracking Key Presses

In order to track key presses, we'll need to update two things simultaneously.
We need to subscribe to key presses in our `subscriptions` function, and we
also need to handle those key presses in our `update` function.

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
different subscriptions, so we could just add a comma and subscribe to mouse
events as well when we need to. For now, all we need to know is that we're
using the Elm Architecture to subscribe to keyboard input via the `presses`
function, and we're going to handle these presses with the `KeyPress` message
in the `update` function.

...

## Viewing Changes

...

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