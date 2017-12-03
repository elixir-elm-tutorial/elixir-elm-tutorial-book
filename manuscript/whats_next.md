# What's Next?

The original plan for this book included additional features and more minigame
ideas. However, the priority shifted towards shipping a solid, stable _v1.0_
release instead of an exhaustive resource. Rather than having a dozen
half-written and half-implemented features, the hope is that this release will
be a stable foundational resource.

What features do we have to look forward to for _v2.0_?

## Additional Platformer Features

The MVP for the Platformer game was to have keyboard interaction to move the
character around on the screen, to be able to collect items, and to increment a
player score accordingly.

## More Minigames

This is the most fun part of the Elixir and Elm Tutorial, and perhaps the
biggest differentiator between this book and many of the other great resources
available.

The intention for this book has been that the games we build should be fun and
interactive. So the focus would be on small classic game genres that involve
keyboard or mouse interaction to move a character around (as opposed to games
like hangman or number guessing or memory games).

The original idea was to implement very small games where players could score
points simply by moving a character around or jumping. But the initial game
grew to increase the fun factor, and the plan is to incorporate quite a few
game elements to make for a slightly more substantial example in a single game.

It would be great to add at least one more game in addition to the initial
example. And the next game would likely incorporate more mouse interaction, so
it could end up being a simple flying/driving game, or potentially be something
more akin to an adventure game where the character can move in more directions.

- Adventure
- Pong
- Snake

Future versions of this book will most certainly contain additional minigame
chapters.

- [404 Elm Street](https://github.com/zalando/elm-street-404)
- [Asteroids](https://github.com/justinmimbs/asteroids)
- [Elm Shooter](https://github.com/sporto/elm-shooter)
- [Vessel](https://github.com/slawrence/vessel)

## Multiplayer?

- [Phoenix Elm Battleship](https://github.com/bigardone/phoenix-elm-battleship)

## Extensible Game Creation

Ideally, we'll make it easy for readers to create their own Elm games and
submit them to the platform. This would allow us to have a library of shared
demos and new readers could build on ideas to create more substantial games.

## OTP Content

OTP is such a big part of what makes Elixir amazing, and it's a travesty not to
have some introductory content in the book. The next release will likely
extract the scoring features into a small GenServer service.

## Chat

A player chatroom should be relatively simple to add to the main page of the
platform. And this may be a good opportunity to provide additional practice and
familiarity with GenServer concepts.
