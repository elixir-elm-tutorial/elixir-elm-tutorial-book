# Phoenix authentication

In the last chapter, we managed to update our players with all the fields we'll
need for authentication. Now we can go ahead and implement our authentication
features.

## Fetching Dependencies

...

## Database Seeds

Now that we have all the fields we want to work with, it can be helpful to add
some default data seeds with our application. Instead of manually creating new
database records while we're working in the development environment, this gives
us a quick way to seed the database.

The other benefit of this approach is that when other developers clone our
repository, they'll be able to run the `mix ecto.setup` command and it will
create the database, run migrations, and seed the application with some sample
data.
