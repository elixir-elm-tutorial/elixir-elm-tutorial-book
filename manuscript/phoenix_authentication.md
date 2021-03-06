# Phoenix Authentication

In the last chapter, we managed to update our players with the fields we'll
need to continue. Now, we can start implementing our authentication features.
We're going to begin with the bare minimum so users can sign up and sign in to
our platform, but we're not going to worry about more advanced authentication
features like email verification or forgotten password mailers. Our goal is to
allow users to sign up and sign in quickly, easily, and securely.

## Fetching Dependencies

To get started with Phoenix authentication, we'll need to add a couple of new
dependencies. At the root of our project, take a look at the `mix.exs` file and
find the `deps/0` function. This function is where we specify which
dependencies our application requires, and we can see there are already a
handful that Phoenix includes by default.

```elixir
defp deps do
  [
    {:phoenix, "~> 1.4.0"},
    {:phoenix_pubsub, "~> 1.1"},
    {:phoenix_ecto, "~> 4.0"},
    {:ecto_sql, "~> 3.0"},
    {:postgrex, ">= 0.0.0"},
    {:phoenix_html, "~> 2.11"},
    {:phoenix_live_reload, "~> 1.2", only: :dev},
    {:gettext, "~> 0.11"},
    {:jason, "~> 1.0"},
    {:plug_cowboy, "~> 2.0"}
  ]
end
```

The dependency we'll use for securing our passwords is called (somewhat
ironically) [comeonin](https://hex.pm/packages/comeonin). This is what the
syntax looks like for adding a new dependency:

```elixir
{:comeonin, "~> 4.1"}
```

In Elixir, this syntax is called a **tuple**. It's commonly used as a way to
reference keys and values. In this example, the first element of the tuple is
an atom (`:comeonin`), and the second element is a string that indicates the
version number (`"~> 4.1"`).

Comeonin allows us to choose from different password hashing algorithms, so
we'll also need to import another dependency called
[bcrypt_elixir](https://hex.pm/packages/bcrypt_elixir) to get everything
working. Let's update our `deps/0` function with the following:

```elixir
defp deps do
  [
    {:phoenix, "~> 1.4.0"},
    {:phoenix_pubsub, "~> 1.1"},
    {:phoenix_ecto, "~> 4.0"},
    {:ecto_sql, "~> 3.0"},
    {:postgrex, ">= 0.0.0"},
    {:phoenix_html, "~> 2.11"},
    {:phoenix_live_reload, "~> 1.2", only: :dev},
    {:gettext, "~> 0.11"},
    {:jason, "~> 1.0"},
    {:plug_cowboy, "~> 2.0"},
    {:comeonin, "~> 4.1"},
    {:bcrypt_elixir, "~> 1.1"}
  ]
end
```

Save that file, and then from the command line we'll run the `mix` command that
fetches dependencies:

```shell
$ mix deps.get
```

We should see the following results:

```shell
$ mix deps.get
Resolving Hex dependencies...
Dependency resolution completed:
...
New
  bcrypt_elixir 1.1
  comeonin 4.1
  ...
* Getting comeonin (Hex package)
* Getting bcrypt_elixir (Hex package)
```

## Player Changesets

Now that we've included our new dependencies, let's take a look at the existing
`changeset/2` function inside the `lib/platform/accounts/player.ex` file.

```elixir
def changeset(player, attrs) do
  player
  |> cast(attrs, [:display_name, :password, :score, :username])
  |> validate_required([:username])
  |> unique_constraint(:username)
end
```

This is where we can add additional validations for our data and ensure that it
conforms to our expectations. This function will remain our default player
changeset, and we'll also add a separate one called `registration_changeset/2`
for when players create a new account.

Let's add some validations and a new function that will allow us to encrypt
passwords so they're not stored in plain text. Update the `changeset/2` function
and add the following code:

```elixir
@doc false
def changeset(player, attrs) do
  player
  |> cast(attrs, [:display_name, :password, :score, :username])
  |> validate_required([:username])
  |> unique_constraint(:username)
  |> validate_length(:username, min: 2, max: 100)
  |> validate_length(:password, min: 2, max: 100)
  |> put_pass_digest()
end

@doc false
def registration_changeset(player, attrs) do
  player
  |> cast(attrs, [:password, :username])
  |> validate_required([:password, :username])
  |> unique_constraint(:username)
  |> validate_length(:username, min: 2, max: 100)
  |> validate_length(:password, min: 2, max: 100)
  |> put_pass_digest()
end

defp put_pass_digest(changeset) do
  case changeset do
    %Ecto.Changeset{valid?: true, changes: %{password: pass}} ->
      put_change(changeset, :password_digest, Comeonin.Bcrypt.hashpwsalt(pass))

    _ ->
      changeset
  end
end
```

With this code, we're able to add a couple of quick validations to ensure data
is structured properly. We're making sure that users enter both a `username`
and `password` when they create a new account, and that those fields are of a
certain length. More importantly, we're piping into our new `put_pass_digest/1`
function, which will encrypt passwords using the `comeonin` dependency.

Our new `put_pass_digest/1` function takes in the player `changeset`, and then
we add a `case` statement to determine whether or not it is valid. If the
`changeset` is valid, we're using our new dependency to hash the `password`
field with `Comeonin.Bcrypt.hashpwsalt(pass)`, and then store the hash in the
`password_digest` field using the
[`put_change/3`](https://hexdocs.pm/ecto/Ecto.Changeset.html#put_change/3)
function. This is the reason we set the `password` field to `virtual: true` in
the player schema, because we're only going to store the hash in the
`player_digest` field.

In the event that the `changeset` was not valid, we just return it at the
bottom of our `put_pass_digest/1` function without any changes. This is a
common pattern we can use for `case` statements where `_` is a useful default
case if none of the branches above applied.

There is admittedly some code duplication between the two changeset functions.
This is primarily due to the fact that we're allowing users to sign up with a
`username` and `password`, and we're also allowing them to change their
`password` field when they edit an account. But the main idea is that we can
have different changesets that apply to different situations. In our case, we
wanted to use the `registration_changeset/2` for when players sign up, and use
a separate `changeset/2` function for any other player changes.

## Using Our New Changeset

To get these features working, we'll need to adjust the `create_player/1`
function in the `lib/platform/accounts/accounts.ex` file. In the other
functions, we'll continue using the `changeset/2` function, but in this one we
want to use the `registration_changeset/2` function so that both the `username`
and `password` fields are required to create an account.

Update the `create_player/1` function with the following:

```elixir
def create_player(attrs \\ %{}) do
  %Player{}
  |> Player.registration_changeset(attrs)
  |> Repo.insert()
end
```

## Accounts Tests and Module Attributes

When we run our tests, we use different sets of attributes to simulate valid
and invalid data. When we ran the generator command to create our players
resource, Phoenix created some initial values for us. We've since updated the
fields that we're working with, so we'll need to make some changes to our
tests as well.

Let's open the `test/platform/accounts/accounts_test.exs` file and take a look
at the attributes:

```elixir
describe "players" do
  alias Platform.Accounts.Player

  @valid_attrs %{score: 42, username: "some username"}
  @update_attrs %{score: 43, username: "some updated username"}
  @invalid_attrs %{score: nil, username: nil}

  # ...
end
```

In Elixir, these are called
[module attributes](https://elixir-lang.org/getting-started/module-attributes.html).
They're useful for creating constants that we can use throughout our tests. For
example, we assign a map of valid player fields to the `@valid_attrs` module
attribute, and then we can use those fields in the tests below.

Let's make some changes to our attribute data to account for the changes we've
made to our player schema:

```elixir
@valid_attrs %{password: "some password", username: "some username"}
@update_attrs %{
  display_name: "some updated display name",
  password: "some updated password",
  score: 43,
  username: "some updated username"
}
@invalid_attrs %{password: nil, username: nil}
```

We want to ensure that our players can sign up with just a `username` and
`password`, so we include those in our `@valid_attrs`. Then, we can ensure that
our other fields work by including them in `@update_attrs`. Lastly, we ensure
that `nil` values won't work to create new accounts by including them in
`@invalid_attrs`.

Let's also go ahead and update our `create_player/1` test case with the
following to since we want users to create accounts with valid `username` and
`password` fields.

```elixir
test "create_player/1 with valid data creates a player" do
  assert {:ok, %Player{} = player} = Accounts.create_player(@valid_attrs)
  assert player.password == "some password"
  assert player.username == "some username"
end
```

## Fixtures, Maps, and Structs

In the same `test/platform/accounts/accounts_test.exs` file, we have a
`player_fixture/1` function that returns a player "struct" we use as a sample
player throughout the test cases. We still want to use this function to create
a sample player, but we also want to ignore the `password` field in our test
environment. We can use this as an opportunity to learn a little bit about
Elixir structs and maps.

We saw some examples of
[Elixir maps](https://elixir-lang.org/getting-started/keywords-and-maps.html#maps)
in the previous section about attributes. They are useful as key-value stores,
and they're considered an essential data structure. Here's a simple example with
two keys and two values:

```elixir
%{password: "some password", username: "some username"}
```

[Elixir structs](https://elixir-lang.org/getting-started/structs.html) are
similar to maps, but have additional structure for defining keys and values.
Here's an example of what a simple player struct might look like:

```elixir
%Platform.Accounts.Player{password: "some password", username: "some username"}
```

Our actual player struct is more complicated, because it needs to account for
all the other fields, including things like `id`, `updated_at`, and other
metadata.

Let's take a look at our existing `player_fixture/1` function:

```elixir
def player_fixture(attrs \\ %{}) do
  {:ok, player} =
    attrs
    |> Enum.into(@valid_attrs)
    |> Accounts.create_player()

  player
end
```

This function takes our map of valid player attributes (`@valid_attrs`) and
uses the `Accounts.create_player()` function to create a new player account.
Then, we pattern match the result to get the `player` struct and return the
`player` at the bottom.

We're going to make a slight change to remove the `password` field. We'll need
to convert the struct into a map to delete the field, and then we'll merge the
fields back together to return the player struct.

```elixir
def player_fixture(attrs \\ %{}) do
  {:ok, player} =
    attrs
    |> Enum.into(@valid_attrs)
    |> Accounts.create_player()

  player_attrs_map =
    player
    |> Map.from_struct()
    |> Map.delete(:password)

  Map.merge(%Player{}, player_attrs_map)
end
```

Rather than just returning the `player` struct at the bottom of the function,
we're converting the struct to a map using `Map.from_struct()` and then
deleting the `password` field with `Map.delete(:password)`. This gives us a map
of all the player attributes except the `password` field.

At the bottom of the function, we use `Map.merge(%Player{}, player_attrs_map)`
to merge all the fields together and convert the results back into a player
struct, which we return as our player fixture.

Keep in mind that this is a lot to take in as we're dealing with new data
structures and a lot of new functions, so don't worry if this seems slightly
overwhelming at first. Working with maps and structs is so common in Elixir and
Phoenix applications that we'll pick it up easily as we gain experience. In the
meantime, this provided a good starting point while we work towards our goal of
fixing our test suite.

## Player Controller Tests

We've updated our player `changeset/2` function and fixed the tests for our
player accounts, but we still have a few failing tests. Let's switch to the
`test/platform_web/controllers/player_controller_test.exs` file and make some
changes.

Similar to the way we updated our module attributes in the previous sections,
let's change `@create_attrs`, `@update_attrs`, and `@invalid_attrs` with the
following:

```elixir
@create_attrs %{password: "some password", username: "some username"}
@update_attrs %{
  display_name: "some updated display name",
  password: "some updated password",
  score: 43,
  username: "some updated username"
}
@invalid_attrs %{password: nil, username: nil}
```

Now that we've made changes to our player schema and adjusted our attributes,
we should be able to run our tests again and see them all passing:

```shell
$ mix test
....................

Finished in 4.4 seconds
19 tests, 0 failures

Randomized with seed 77808
```

## Speeding Up Tests

This part is optional, but it provides a good example of how we can configure
our test environment and speed things up. You may have noticed that our tests
are running more slowly than they were before. The password hashing algorithm
takes a while, and we don't necessarily need this in our test environment.

Let's add a configuration setting at the bottom of our `config/test.exs` file:

```elixir
# Reduce bcrypt rounds to speed up tests
config :bcrypt_elixir, :log_rounds, 4
```

Let's save the file and run our tests again to see if there's a difference:

```shell
$ mix test
Compiling 20 files (.ex)
Generated platform app
....................

Finished in 0.2 seconds
20 tests, 0 failures

Randomized with seed 749042
```

We reduced the number of encryption rounds our hashing algorithm is running
(only in our test environment), and this resulted in a noticeable difference in
the amount of time our tests took to run (from 4.4 seconds to 0.2 seconds).

## Authentication Plug

Players are currently able to create new accounts at
`http://localhost:4000/players/new`. But we'll want to add features so that users
can sign in and sign out.

![Player Sign Up Page](images/phoenix_sign_up/phoenix_updated_sign_up.png)

Let's make a new controller called `PlayerAuthController`. Create a
`lib/platform_web/controllers/player_auth_controller.ex` file, and add the
following content:

```elixir
defmodule PlatformWeb.PlayerAuthController do
  import Plug.Conn

  alias Platform.Accounts.Player

  def init(opts) do
    Keyword.fetch!(opts, :repo)
  end

  def call(conn, repo) do
    player_id = get_session(conn, :player_id)
    player = player_id && repo.get(Player, player_id)
    assign(conn, :current_user, player)
  end
end
```

This will allow us to collect information about the current player's session
and assign it to `:current_user` so we can refer to that when handling our
authentication features.

## Router

Remembering back to when we set up our `PlayerController` in the Phoenix
router, we used the default browser pipeline. If we open the
`lib/platform_web/router.ex` file, we'll see that there are quite a few
`plug`s at the top:

```elixir
defmodule PlatformWeb.Router do
  use PlatformWeb, :router

  pipeline :browser do
    plug :accepts, ["html"]
    plug :fetch_session
    plug :fetch_flash
    plug :protect_from_forgery
    plug :put_secure_browser_headers
  end

  # ...
end
```

At the bottom of the `pipeline :browser` block, let's add our new
authentication plug:

```elixir
pipeline :browser do
  plug :accepts, ["html"]
  plug :fetch_session
  plug :fetch_flash
  plug :protect_from_forgery
  plug :put_secure_browser_headers
  plug PlatformWeb.PlayerAuthController, repo: Platform.Repo
end
```

This plug is going to allow us to restrict access to certain pages. Let's
update our application so users will be redirected to the **New Player** page
at `http://localhost:4000/players/new` if they haven't already signed in.

Currently, when we access `http://localhost:4000` in the browser, we see the
index page from our `PageController`. In the upcoming chapters, we'll be
turning this into our Elm front-end application.

![Home Page](images/diving_in/updated_home_page.png)

We're going to use our new `PlayerAuthController` to redirect users to the
**New Player** page before they can view our home page.

## Authenticate Function

Currently, users are able to navigate to all the pages within our application.
We want to ensure that players are signed in before they access our Elm
application and start playing games.

At the bottom of our `PageController`, let's add an `authenticate/2`
function. Open up the `lib/platform_web/controllers/page_controller.ex` file
and add the following beneath the `index/2` function:

```elixir
defp authenticate(conn, _opts) do
  if conn.assigns.current_user() do
    conn
  else
    conn
    |> put_flash(:error, "You must be signed in to access that page.")
    |> redirect(to: Routes.player_path(conn, :new))
    |> halt()
  end
end
```

Since we assigned the current player's session to be the `current_user` inside
our `PlayerAuthController`, we can use that to determine whether a visitor to
our site is signed in. If they are, we'll just return the connection and allow
them to continue. Otherwise, if they're attempting to access a restricted
resource, we'll display a message and redirect them back to the **New Player**
page.

In the same file, we'll use the `authenticate/2` function that we just created
to determine whether or not players should be able to render the
`PageController` index page.

Above the `index/2` function, add the following line of code:

```elixir
plug :authenticate when action in [:index]
```

This is what the full `lib/platform_web/controllers/page_controller.ex` file
should look like:

```elixir
defmodule PlatformWeb.PageController do
  use PlatformWeb, :controller

  plug :authenticate when action in [:index]

  def index(conn, _params) do
    render conn, "index.html"
  end

  defp authenticate(conn, _opts) do
    if conn.assigns.current_user() do
      conn
    else
      conn
      |> put_flash(:error, "You must be signed in to access that page.")
      |> redirect(to: Routes.player_path(conn, :new))
      |> halt()
    end
  end
end
```

## Manual Testing

If you're wondering if the updates above broke our tests, you're right. We
usually run our test suite with `mix test`, but this time let's manually test
things out in the browser. Open up your browser and try visiting the
`PageController` index page at `http://localhost:4000`:

![Restricted Page Alert](images/phoenix_authentication/restricted_page.png)

It looks like our changes worked, because the application redirected us back to
the **New Player** page. We managed to restrict access to the index page, and
we see the flash alert ("You must be signed in to access that page.") that we
wrote in the `authenticate/2` function at the bottom of the `PageController`.

## Fixing Our Tests

Let's push a quick fix for our tests. Open the
`test/platform_web/controllers/page_controller_test.exs` file and replace it
with the following code that tests our default route and our new redirect:

```elixir
defmodule PlatformWeb.PageControllerTest do
  use PlatformWeb.ConnCase

  test "redirects unauthenticated users away from index page", %{conn: conn} do
    conn = get(conn, "/")
    assert html_response(conn, 302) =~ "redirect"
  end
end
```

Go ahead and run the `mix test` command again, and we should be all set with
passing tests that give us confidence to keep adding features.

## Signing In

How do we allow users to sign in to their newly created accounts? Let's define
a `sign_in/2` function in our `PlayerAuthController`:

```elixir
def sign_in(conn, player) do
  conn
  |> assign(:current_user, player)
  |> put_session(:player_id, player.id)
  |> configure_session(renew: true)
end
```

Right after a new player creates an account, we automatically want to use the
`sign_in/2` function to authenticate them on our platform. To accomplish this,
we'll need to update the `create/2` function in our `PlayerController`. We'll
use the pipe operator to sign the player in before we display the flash message
and redirect them:

```elixir
def create(conn, %{"player" => player_params}) do
  case Accounts.create_player(player_params) do
    {:ok, player} ->
      conn
      |> PlatformWeb.PlayerAuthController.sign_in(player)
      |> put_flash(:info, "Player created successfully.")
      |> redirect(to: Routes.player_path(conn, :show, player))

    {:error, %Ecto.Changeset{} = changeset} ->
      render(conn, "new.html", changeset: changeset)
  end
end
```

Let's try it out. Go to the **New Player** page at
`http://localhost:4000/players/new` and create a new user with both the
`username` and `password` fields set to `chrismccord`:

![Creating a New User](images/phoenix_authentication/new_user_to_sign_in.png)

The new user we just created should now be signed in, and we'll be directed to
the **Show Player** page for the new user.

![Show Player Page](images/phoenix_authentication/show_player_page_after_sign_in.png)

Now, let's try to access the `PageController` index page to verify that the
user is authenticated. Go to the `http://localhost:4000` page in your browser:

![Signed In Access to Index Page](images/diving_in/updated_home_page.png)

Success! We're able to access this page because we created a new account and
authenticated the new player at the same time.

## Sessions

To complete our authentication features, we'll need to handle user sessions. We
were able to handle the sign in feature in the previous section because we took
care of it while creating an account, but now we'll also want to allow users to
sign out and sign back in whenever they'd like.

Let's start by creating a `PlayerSessionController`. Create a new file called
`lib/platform_web/controllers/player_session_controller.ex`, and add the
following code:

```elixir
defmodule PlatformWeb.PlayerSessionController do
  use PlatformWeb, :controller

  def new(conn, _) do
    render(conn, "new.html")
  end

  def create(conn, %{"session" => %{"username" => user, "password" => pass}}) do
    case PlatformWeb.PlayerAuthController.sign_in_with_username_and_password(
           conn,
           user,
           pass,
           repo: Platform.Repo
         ) do
      {:ok, conn} ->
        conn
        |> put_flash(:info, "Welcome back!")
        |> redirect(to: Routes.page_path(conn, :index))

      {:error, _reason, conn} ->
        conn
        |> put_flash(:error, "Invalid username/password combination.")
        |> render("new.html")
    end
  end

  def delete(conn, _) do
    conn
    |> PlatformWeb.PlayerAuthController.sign_out()
    |> redirect(to: Routes.player_session_path(conn, :new))
  end
end
```

This may seem like a lot at first, but we'll go through it quickly along with
the updates we make to the `PlayerAuthController`.

The `new/2` function will allow us to display a **Player Sign In** page (as
opposed to the **New Player** page that new players will use). When we create
new sessions, we'll use a new function (which we'll create soon) called
`sign_in_with_username_and_password/4`. Lastly, the `delete/2` function will
allow users to sign out by deleting their session.

## Player Sign In View and Template

Let's create the view and the template for our player **Player Sign In** page.
Add a new file called `player_session_view.ex` inside the
`lib/platform_web/views` folder. Then, add the following to the file:

```elixir
defmodule PlatformWeb.PlayerSessionView do
  use PlatformWeb, :view
end
```

We'll also need to create the corresponding template. Create a
`lib/platform_web/templates/player_session` folder, and then add a
`new.html.eex` file inside with the following content:

```embedded_elixir
<h1>Player Sign In</h1>

<%= form_for @conn, Routes.player_session_path(@conn, :create), [as: :session], fn f -> %>
  <%= label f, :username %>
  <%= text_input f, :username %>
  <%= error_tag f, :username %>

  <%= label f, :password %>
  <%= password_input f, :password %>
  <%= error_tag f, :password %>

  <div>
    <%= submit "Sign In" %>
  </div>
<% end %>
```

This gives us a **Player Sign In** page with a form for existing users to enter
their `username` and `password`.

## Session Routing

If you were still running a Phoenix server this whole time, you've probably
noticed we've been creating errors in our application. But we're getting close
to a working authentication system. We'll need to update our router to reflect
our new session features. Open the `lib/platform_web/router.ex` file and add
our session resource:

```elixir
scope "/", PlatformWeb do
  pipe_through :browser

  get "/", PlayerController, :new
  resources "/players", PlayerController
  resources "/sessions", PlayerSessionController, only: [:new, :create, :delete]
end
```

This will add the necessary routes so our players can create and delete their
sessions to sign in and out of the platform.

## Signing In and Out

Lastly, we'll create the functions in our `PlayerAuthController` that tie
everything together. Add the `sign_in_with_username_and_password/4` function and
the `sign_out/1` function below the `sign_in/2` function at the bottom of the
`lib/platform_web/controllers/player_auth_controller.ex` file.

```elixir
def sign_in_with_username_and_password(conn, username, given_pass, opts) do
  repo = Keyword.fetch!(opts, :repo)
  player = repo.get_by(Player, username: username)

  cond do
    player && Comeonin.Bcrypt.checkpw(given_pass, player.password_digest) ->
      {:ok, sign_in(conn, player)}

    player ->
      {:error, :unauthorized, conn}

    true ->
      Comeonin.Bcrypt.dummy_checkpw()
      {:error, :not_found, conn}
  end
end

def sign_out(conn) do
  configure_session(conn, drop: true)
end
```

What the `sign_in_with_username_and_password/4` function does is to grab the
player from the database by their `username` field. If the player exists and
has the correct password, they will be signed in. Otherwise, we'll return an
error.

For the `sign_out/1` function, we're dropping the current user's session to
sign them out of the platform.

## Trying Things Out

It's a good idea to try out these features using "incognito" browser windows.
That will give us a way to open a new browser window in a clean state. If
you're using Google Chrome on macOS, you can create a new incognito window with
`Command + Shift + N`. It's also a good idea to restart your Phoenix server
with `mix phx.server` at this point to get things up and running.

We can test out the sign in process with the same account we created in the
previous sections. Go to the **Player Sign In** page at
`http://localhost:4000/sessions/new` and try entering `chrismccord` for both the
`username` and `password` fields.

![Player Sign In Page](images/phoenix_authentication/player_sign_in_page.png)

Success! We get a "Welcome back!" message letting us know that we were able to
sign in successfully:

![Successful Sign In](images/phoenix_authentication/successful_sign_in_message.png)

## Displaying the Player Status

We've given users the ability to create new accounts at `/players/new`, and
players can sign in with existing accounts at `/sessions/new`. As a last step
for this chapter, let's find a place to let players know when they're signed
in, and give them a way to sign out.

First, let's open the `lib/platform_web/templates/layout/app.html.eex` file and
remove the default Phoenix header.

Update the contents of the `<header>` tag with the following:

```embedded_elixir
<header>
  <section class="container">
    <nav role="navigation">
      <ul>
        <%= link "Sign Up", to: Routes.player_path(@conn, :new), class: "button" %>
        <%= link "Sign In", to: Routes.player_session_path(@conn, :new), class: "button" %>
      </ul>
    </nav>
    <a href="https://phoenixframework.org/" class="phx-logo">
      <img src="<%= Routes.static_path(@conn, "/images/phoenix.png") %>" alt="Phoenix Framework Logo"/>
    </a>
  </section>
</header>
```

This places buttons in the header for users to either sign up or sign in.

![Header Sign Up and Sign In Buttons](images/phoenix_authentication/authentication_buttons.png)

Depending on whether players are signed in or not, we want to show something
different in the header. Before a player has authenticated, we want to display
the buttons that we see in the screenshot above. If a player has signed in, we
want to show them their `username` in the header along with a sign out button.

Let's update our `<header>` tag again with the following:

```embedded_elixir
<header>
  <section class="container">
    <nav role="navigation">
      <ul>
        <%= if @current_user do %>
          <p>Signed in as <strong><%= @current_user.username %></strong></p>
          <%= link "Sign Out", to: Routes.player_session_path(@conn, :delete, @current_user), method: "delete", class: "button" %>
        <% else %>
          <%= link "Sign Up", to: Routes.player_path(@conn, :new), class: "button" %>
          <%= link "Sign In", to: Routes.player_session_path(@conn, :new), class: "button" %>
        <% end %>
      </ul>
    </nav>
    <a href="https://phoenixframework.org/" class="phx-logo">
      <img src="<%= Routes.static_path(@conn, "/images/phoenix.png") %>" alt="Phoenix Framework Logo"/>
    </a>
  </section>
</header>
```

In the code above, we're creating a conditional to determine whether or not a
`@current_user` is present. When there is a user signed in, we display their
`username` with small text along with a sign out button.

![Header Signed In Display](images/phoenix_authentication/signed_in_header.png)

## Summary

We managed to accomplish what we set out to do in this chapter. We've given our
users a simple way to sign up, sign in, and sign out of our application. We
also have the ability to restrict access to certain pages to ensure that users
are authorized to access the correct data. Next, we'll be building out the
games section for our platform, and getting our first taste of what it's like
to work with Elm.
