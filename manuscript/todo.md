# TODO

## Big Picture Steps Remaining

- [ ] Game levels and player skill acquisition.
- [ ] Channels for syncing gameplay data.

## Bugs

- [ ] Gracefully handle null values when decoding JSON into Elm front-end.
- [ ] Still need to cast other fields in `registration_changeset` or seeds
      don't work: `cast(attrs, [:display_name, :password, :score, :username])`.
- [ ] If I had it to do over again, I think I'd move `elm-package.json` and
      `elm-stuff` into the `assets/elm` folder. If I have time, I might try to
      rewrite that content from the ground up.

## Players

- [ ] Fix **Player Sign In** form with labels and errors.
- [ ] Restrict access to **Listing Players** page after working API.
- [ ] Restrict access to **Edit Player** page to current player.
- [ ] Add *Delete Account* button to **Edit Player** edit page.
- [ ] Remove extra buttons from **Sign Up** and **Sign In** pages.

## Games

- [ ] Add `slug` field from the beginning?

## Gameplays

- [ ] ...

## Elm

- [ ] ...

## Chores

- [ ] Run `mix hex.outdated` and verify that content still works with latest.
