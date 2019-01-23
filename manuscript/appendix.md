# Appendix {#appendix}

## Quick Install

This book is intended for developers with some previous experience, so
installing these languages and tools shouldn't be overly difficult or
time-consuming. Having said that, it's easy to get tripped up with installation
and configuration steps, so feel free to
[create a GitHub issue](https://github.com/elixir-elm-tutorial/elixir-elm-tutorial-book/issues)
if you think there's an easier approach to setting things up.

The intention for this chapter is to get everything we'll need installed
quickly so we can start creating Phoenix projects.

These instructions assume that you're running macOS, but instructions can also
be found online for installing these tools on Linux.

### Elixir

First, let's install [Elixir](https://elixir-lang.org) with
[Homebrew](https://brew.sh). This command will also install the latest version
of [Erlang](https://www.erlang.org) as a dependency:

```shell
$ brew install elixir
```

You can verify that Elixir has been installed properly by running the following
command:

```shell
$ elixir -v
Erlang/OTP 20
Elixir 1.7.0
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

[Phoenix](https://phoenixframework.org) is a web application framework built
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

We'll be using [PostgreSQL](https://www.postgresql.org) for our database. The
easiest way to get started if you're new to PostgreSQL is to use
[**Postgres.app**](https://postgresapp.com). It's a macOS application that
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

Throughout this book, we opt for a simple approach to afford ourselves an
opportunity to learn about Elixir, Phoenix, and Elm as we put together a demo
application. As you start to develop more involved projects, it's a good idea
to review additional tools and services that can make your life easier.

The [hex.pm](https://hex.pm) package manager is an invaluable tool for finding
useful libraries for your projects. For example, if you want to allow your
users to write Markdown syntax, you can look up "Markdown" on hex.pm and find
that the [Earmark](https://github.com/pragdave/earmark) package works really
well for this.

Listed below are additional tools for your consideration.

### Authentication

Want to build more robust authentication features for your application?
Consider checking out the following options:

- [ueberauth](https://github.com/ueberauth/ueberauth)
- [guardian](https://github.com/ueberauth/guardian)

### Authorization

We briefly touched on authorizing actions in our demo application. If you need
to work with additional authorization policies, take the following into
consideration:

- [BodyGuard](https://github.com/schrockwell/bodyguard)

### Code Quality

Credo is a code quality tool that performs static analysis on your Elixir code
and provides helpful tips and feedback. It's really helpful as a way to learn
solid Elixir conventions and keep the code throughout your project consistent.

- [Credo](https://github.com/rrrene/credo)

### Documentation

Elixir has amazing documentation tools, which explains why the docs are so
fantastic. Check out the Elixir guide on
[writing documentation](https://hexdocs.pm/elixir/writing-documentation.html)
and consider using `ExDoc` to generate docs for your project.

- [ExDoc](https://github.com/elixir-lang/ex_doc)

### Continuous Integration

Early versions of this book included material on Continuous Integration and
Continuous Delivery. Although it ended up being outside the scope of our
content, it's essential to have a CI server to automatically run your tests.
Check out the following options, and consider hooking them into your GitHub
repository to automatically deploy your application.

- [CircleCI](https://circleci.com)
- [Semaphore](https://semaphoreci.com)

### Monitoring

Once your project is deployed to production, it's a good idea to monitor the
performance and watch for errors. AppSignal is a good option for tracking this
data and keeping your application running smoothly.

- [AppSignal](https://appsignal.com/elixir)

### Testing

Because we used the Phoenix generators to scaffold out our initial features,
our demo application came with quite a few tests. So we have examples of how
to work with `ExUnit` in our project, but Wallaby is a great option for writing
highly readable integration tests concurrently.

- [Wallaby](https://github.com/keathley/wallaby)
