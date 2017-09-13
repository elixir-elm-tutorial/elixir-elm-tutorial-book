# TODO

## Big Picture

- [ ] Routing for games.
- [ ] Channels for syncing gameplay data.
- [ ] Game levels and player skill acquisition.

## Bugs

- [ ] Still need to cast other fields in `registration_changeset` or seeds
      don't work.
  - `cast(attrs, [:display_name, :password, :score, :username])`
- [ ] Gracefully handle null values when decoding JSON.
- [ ] If I had it to do over again, I think I'd move `elm-package.json` and
      `elm-stuff` into the `assets/elm` folder. If I have time, I might try to
      rewrite that content from the ground up.

## Players

- [ ] Fix **Player Sign In** form with labels and errors.
- [ ] Restrict access to players index page after working API.
- [ ] Add link to **Edit Player** page in header.
- [ ] Restrict accesss to player edit page to current player.
- [ ] Add delete player button to player edit page.
- [ ] Remove extra buttons from **Sign Up** and **Sign In** pages.
- [ ] Change `score` field for players to `total_score`?
- [ ] Add a string URL `avatar` field for players?

## Games

- [ ] Add `slug` field from the beginning?

## Gameplays

- [ ] ...

## Elm

- [ ] ...

## Chores

- [ ] Run `mix hex.outdated` and verify that content still works with latest.