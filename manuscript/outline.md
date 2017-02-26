# Technical Outline

## Building a Game Platform

Using Elixir and Phoenix to build the API for a game platform, and then use Elm
to build a mini game.

### Installation Requirements

- brew install erlang elixir
- npm
- PostgreSQL

### Creating the Phoenix App

- mix phoenix.new platform
- cd platform
- mix ecto.create
- mix test
- mix phoenix.server
- [http://localhost:4000](http://localhost:4000)

### Creating Players

- mix phoenix.gen.html Player players name:string email:string handle:string
- web/router.ex: resources "/players", PlayerController
- mix ecto.migrate
- mix test
- mix phoenix.server
- [http://localhost:4000/players](http://localhost:4000/players)
- create first player

### Creating Games

### Styles

### JSON API

### Authentication

### Channels

### Building a Chatroom for Players

### Setting Up Elm

* install elm
* elm-format
* elm-brunch

### Building a Small Game with Elm

### Tracking the Score

### Deployment to Heroku

