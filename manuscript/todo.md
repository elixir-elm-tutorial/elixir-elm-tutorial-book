# TODO

## Bugs

- [ ] Still need to cast other fields in `registration_changeset` or seeds don't work.
  - `cast(attrs, [:display_name, :password, :score, :username])`
- [ ] Gracefully handle null values when decoding JSON.

## Big Picture

- [ ] Routing for games.
- [ ] Channels for syncing gameplay data.
- [ ] Game levels and player skill acquisition.

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

- [ ] Add `slug` field?

## Gameplays

- [ ] ...

## Elm

- [ ] Rewrite setup chapter with new location.
- [ ] Add list of players and games on index page.

## Chores

- [ ] Run `mix hex.outdated` and verify that content still works with latest.