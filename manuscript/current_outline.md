# Current Outline

Keep in mind that this is a _very_ early version of the Elixir and Elm
Tutorial! This outline is an attempt to break down all the topics and concepts
that are covered throughout the book. The structure and content is a work in
progress.

- Introduction
  - Metadata about Elixir, Elm, and functional programming.
  - Prerequisites and acknowledgements.
  - Information about the demo application.

- Diving In
  - Quick-paced practical introduction to the Phoenix framework.
  - Building the initial Platform demo application.
  - Configuring the PostgreSQL database, running the server, and routing.
  - Generating the HTML resources for players.

- Elixir Introduction
  - Create a temporary Elixir application to compare of Elixir and Phoenix
    projects.
  - Brief background on mix, folder structure, modules, functions,
    documentation, tests, and interactive environment.
  - Introduction to concepts on piping, arity, pattern matching, and guards.

- Phoenix Testing and Deployment
  - Running Phoenix tests.
  - Working with Git and GitHub.
  - Configuring the application and deploying to Heroku.
  - Continuous integration and deployment with TravisCI and Heroku.

- Phoenix Sign Up
  - Extending existing resources with new fields.
  - Generating and running migrations.
  - Basic queries with IEx.
  - Updating templates and working with forms.
  - Data validation and database seeds.

- Phoenix Authentication
  - Importing Hex dependencies.
  - Working with changesets.
  - Building an authentication plug.
  - Adding login and session features.

- Phoenix API
  - Generating a JSON API for games.

- Elm Introduction
  - Brief introduction to the Elm language and tooling.
  - Covers modules, functions, types, formatting, and refactoring.

- Elm Setup
  - Introduction to Brunch and Phoenix assets.
  - Configuring Phoenix to work with Elm.

- Elm Application
  - Starting our Elm front-end application.
  - Importing and using Elm packages.
  - Working with familiar HTML as a bridge to learning Elm.
  - Breaking up code into small, pure functions.
  - Working with the concept of Maybe in Elm.
  - Iterating through lists of data.

- Elm Architecture
  - Structure of the Elm Architecture.
  - Working with the Record data type in Elm.
  - Adding the Model, Update, Subscriptions, and View.
  - Pulling the application together with the Main function.
  - Adding initial interactivity with Html.Events.

- Elm API Data (Incomplete)
  - Reading JSON API data from Phoenix.
  - Decoding JSON and working with game data in Elm.

- Our First Game
  - Creating a space within Phoenix for an Elm game.
  - Working with SVG to create a small game canvas.
  - Adding a small game character and item.
  - Refactoring Elm code with let expressions and small functions.

- Adding Interaction
  - Working with Elm subscriptions and keyboard input.
  - Adjusting character position.
  - Spawning and collecting items.
  - Working with randomness.

- Displaying Game Data
  - Rendering text in the game window.
  - Displaying the player score, items collected, and time remaining.
  - Working with time.

- Handling Game States
  - Introducing union types for game states.
  - Rendering different elements depending on current game states.
  - Creating start, success, and game over states.

- Movement (Incomplete)
  - Implementing character velocity and updating keyboard interaction.
  - Adjusting directional changes and assets.
  - Adding character running and jumping abilities.

- Changing Levels (Incomplete)
  - Cursory introduction to game theory and design.
  - Switching from randomness to pattern recognition.
  - Adding gradual player skill progression.
  - Implementing level advancement and final success state.

- Syncing Score Data (Coming Soon)
  - Getting started with Phoenix channels.
  - Configuring elm-phoenix-socket.

- Games (Coming Soon)
  - Initial discussion of games and incremental approach.
  - Getting started with game implementation.

- Chat Feature (Time Permitting)
  - This chapter may be added only if time allows.
  - Using Phoenix channels to add a chat lobby for players.

- Score Data (Time Permitting)
  - This chapter may be added only if time allows.
  - Using GenServer to extract score feature into small service.

- Appendix
  - Brief installation instructions.

- Contact
  - Congrats and plea for feedback and ideas.
  - Bonus access to Slack for live help or sharing.
