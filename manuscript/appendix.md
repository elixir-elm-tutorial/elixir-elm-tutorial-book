# Appendix {#appendix}

## Quick Install

This book is intended for developers with some previous experience, so
installing these languages and tools shouldn't be overly difficult or
time-consuming. Having said that, it's easy to get tripped up with installation
and configuration steps, so feel free to
[create a GitHub Issue](https://github.com/elixir-elm-tutorial/elixir-elm-tutorial-book/issues) if you
think there's an easier approach to setting things up.

The intention for this chapter is that we want to get everything we'll need
installed quickly so we can start creating Phoenix projects.

These instructions assume that you're running macOS, but instructions can also
be found online for installing these tools on Linux.

### Elixir

First, let's install [Elixir](https://elixir-lang.org/) with
[Homebrew](https://brew.sh/). This command will also install the latest version
of [Erlang](https://www.erlang.org/) as a dependency:

```shell
$ brew install elixir
```

You can verify that Elixir has been installed properly by running the following
command:

```shell
$ elixir -v
Erlang/OTP 20
Elixir 1.5.2
```

Any trouble with this step? Check out the
[Elixir install page](https://elixir-lang.org/install.html) or the
[Elixir section of Stack Overflow](https://stackoverflow.com/questions/tagged/elixir).

### Hex

[Hex](https://hex.pm) is the package manager for the Elixir and Erlang
ecosystems. Once you have Elixir installed, it's easy to install Hex with the
following command:

```shell
$ mix local.hex
```

Any trouble with this step? Check out the
[Hex section of Stack Overflow](https://stackoverflow.com/questions/tagged/hex-pm).

### Phoenix

[Phoenix](http://phoenixframework.org/) is a web application framework built
with the Elixir language. You can install the latest version with the following
command:

```shell
$ mix archive.install https://github.com/phoenixframework/archives/raw/master/phx_new.ez
```

You can verify that Phoenix has been installed properly by running the
`mix help` command, and you should be able to see a `mix phx.new` task that will
allow us to create new Phoenix applications.

Any trouble with this step? Check out the
[Phoenix installation docs](https://hexdocs.pm/phoenix/installation.html) or the
[Phoenix section of Stack Overflow](https://stackoverflow.com/questions/tagged/phoenix-framework).

### PostgreSQL

We'll be using [PostgreSQL](https://www.postgresql.org/) for our database. The
easiest way to get started if you're new to PostgreSQL is to use
[**Postgres.app**](https://postgresapp.com/). It's a macOS application that
makes it really simple to get PostgreSQL up and running, and also creates a
`postgres` user that Phoenix uses as a default when creating databases.

Any trouble with this step? Check out the
[PostgreSQL detailed installation guides](https://wiki.postgresql.org/wiki/Detailed_installation_guides)
or the [PostgreSQL section of Stack Overflow](https://stackoverflow.com/questions/tagged/postgresql).

## Working with Versions

The steps above should be all that's required to get started. If you're
interested in working with multiple versions different languages,
check out the [asdf version manager](https://github.com/asdf-vm/asdf).

## Recommended Tools

### Authentication

- [ueberauth](https://github.com/ueberauth/ueberauth)

### Authorization

- [BodyGuard](https://github.com/schrockwell/bodyguard)

### Code Quality

- [Credo](https://github.com/rrrene/credo)

### Continuous Integration

- [CircleCI](https://circleci.com)
- [Semaphore](https://semaphoreci.com)

### Monitoring

- [AppSignal](https://appsignal.com/elixir)
- [PryIn](https://github.com/pryin-io/pryin)

### Testing

- [Wallaby](https://github.com/keathley/wallaby)