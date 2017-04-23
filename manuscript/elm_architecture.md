# Elm Architecture

In this chapter, we're going to adapt our existing Elm application so that it
follows the standard Elm Architecture. While Elm is a programming language like
JavaScript, we can think of the Elm Architecture as a framework for building
applications like React or Angular.

The Elm Architecture gives us a standard approach for how to structure our Elm
applications, and makes it really easy to add our initial features.

## Adapting Our Existing Elm Application

The good news is that our existing Elm application already has the elements we
need for the Elm Architecture. We're just going to adapt the existing code and
add a few things that will tie it all together.

One way to think of our Elm application is to break it into five sections:

- Main
- Model
- Update
- Subscriptions
- View

Each of these sections will be a function, and the Elm runtime will make
everything work by tying them together in the `main` function.

- Main: `main`
- Model: `init`
- Update: `update`
- Subscriptions: `subscriptions`
- View: `view`

## Removing the Original Main Function

We're going to be breaking our original Elm application with all the changes
we make in this chapter. But we don't want elm-format to keep warning us of
errors while we're working. Let's comment out our original `main` function so
we can keep working and using elm-format without it trying to build our full
application for us:

```elm
-- MAIN
-- main : Html msg
-- main =
--     gamesIndex model
```

At the end of this chapter, we'll create a new version of the `main` function
that will pull everything together for us.

## Starting with the Model

The Model will be a good place to start as we structure our data. Instead of
just displaying our game titles, let's show both the game title and a brief
description. We're going to be mirroring some of the data we originally created
in our Phoenix JSON API for games.

We'll name this function `initialModel`, and we'll use Elm's "Record" syntax
to create our games:

```elm
initialModel =
    [ { gameTitle = "Adventure Game", gameDescription = "Adventure game example." }
    , { gameTitle = "Driving Game", gameDescription = "Driving game example." }
    , { gameTitle = "Platform Game", gameDescription = "Platform game example." }
    ]
```

Note that we haven't added type annotations yet, and we're working with a new
data type. Records allow us to create keys and values for our fields. In this
example, we have three records (each one is a game), and each record has two
fields (`gameTitle` and `gameDescription`).

Now that we have some idea of how our data will be structured, let's add a
couple of type annotations for our individual games and our model as a whole:

```elm
type alias Model =
    List Game


type alias Game =
    { gameTitle : String
    , gameDescription : String
    }


initialModel : Model
initialModel =
    [ { gameTitle = "Adventure Game", gameDescription = "Adventure game example." }
    , { gameTitle = "Driving Game", gameDescription = "Driving game example." }
    , { gameTitle = "Platform Game", gameDescription = "Platform game example." }
    ]
```

This is our first look at creating type aliases. We're creating one type alias
for our overall `Model`, which is a list of games. Then we define the type of
our `Game` records. Each game is a record that will contain a `gameTitle` field
and a `gameDescription` field, both of which are `String` types. We also add a
type annotation for our `initialModel` function to ensure that it follows the
`Model` type alias structure that we created above.

Let's make a slight change to help further our understanding of what we're
accomplishing here. We'll make our `initialModel` more flexible for adding
other fields later by moving our records into a named list:

```elm
type alias Model =
    { gamesList : List Game
    }


type alias Game =
    { gameTitle : String
    , gameDescription : String
    }


initialModel : Model
initialModel =
    { gamesList =
        [ { gameTitle = "Adventure Game", gameDescription = "Adventure game example." }
        , { gameTitle = "Driving Game", gameDescription = "Driving game example." }
        , { gameTitle = "Platform Game", gameDescription = "Platform game example." }
        ]
    }
```

This is essentially the same data we were already working with. We're just
turning our overarching `Model` into a record so that we can easily add other
fields later (for example, we'll eventually want to add a `playersList`).

Now we have types that hold data for both our list of games and the individual
games themselves. This enables us to ensure that our data is always properly
structured and that we're not producing any errors.


