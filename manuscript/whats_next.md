# What's Next?

The original plan for this book included additional features and more minigame
ideas. However, the priority shifted to shipping a solid, stable release over
an exhaustive resource. Rather than having many partially implemented
features, the hope is that this release will be a stable foundational resource.

As you grow your skills and experience, here are some additional features and
content ideas to consider.

## Additional Platformer Features

The minimum viable product for the *Platformer* game was to:

- move the character around the screen with keyboard interaction
- allow players to collect items
- increment a player score

This worked well for having a small game with a scoring mechanism we could use
to implement a score syncing feature. But there's also a lot more that we could
do with the *Platformer* game example. Features like running, jumping, and
scrolling would be great additions to the game even if they were outside the
scope of this book.

## More Minigames

The idea of adding interactive minigames was perhaps the most fun part of the
*Elixir and Elm Tutorial*, and probably the biggest differentiator between this
book and many of the other great resources available. The intention was that
the games would be fun and interactive, involving keyboard or mouse input as
opposed to guessing games or memory games.

The original plan was to implement tiny games where players could score points
simply by moving a character around or pressing a certain key. But the initial
game example grew to increase the fun factor.

It would be great to add at least one more game to the book. A new game could
incorporate mouse interaction, and it would be fun to implement a simple flying
or driving game.

For additional inspiration, be sure to check out some of the amazing examples of
other games built with Elm:

- [404 Elm Street](https://github.com/zalando/elm-street-404)
- [Asteroids](https://github.com/justinmimbs/asteroids)
- [Elm Pong](https://github.com/ElmOrlando/ElmPong)
- [Elm Shooter](https://github.com/sporto/elm-shooter)
- [Elm Snake](https://github.com/tibastral/elm-snake)
- [Vessel](https://github.com/slawrence/vessel)

## Multiplayer

Another feature that would be great for the game platform and any minigames
would be to incorporate multiplayer. After authentication, we could potentially
use Phoenix channels to build multiplayer features.

This is likely to be a bit more difficult to implement than some of the other
features, but there's a great example available here if you're interested in
pursuing it further:

- [Phoenix Elm Battleship](https://github.com/bigardone/phoenix-elm-battleship)

## Flexible Game Creation

The book's content currently covers the creation of a single minigame example,
and the process of adding it to the platform could conceivably be much easier.

The Platformer game example could also be improved by extracting common game
elements into a separate module. A global `Game` module could mean having a
common game interface with player information, game information, and scores
displayed, and the game itself could be imported within that structure.

Additionally, if the process of contributing a game were straightforward, then
it would be amazing to encourage all readers of this book to share their own
creations and create a small library of games to try out.

## Chat

A player chatroom should be relatively simple to add to the main page of the
platform. And this may be a good opportunity to provide additional practice and
familiarity with GenServer concepts.

## OTP Content

OTP is such a big part of what makes Elixir amazing, and it's a travesty not to
have some introductory content in the book. One possibility would be to extract
the scoring features into a small GenServer service.

## Summary

Although we didn't get to cover everything we hoped in this book, we managed to
make a _ton_ of progress and accomplish our goal of building a real-world
project together.

The plan is that this book will be continually updated to stay relevant with
new versions of Elixir, Elm, and Phoenix. And it will be exciting to
continually hone the content and add additional material.
