# Introduction

Welcome to the world of functional web programming! In this book, we'll **learn
how to create fun, scalable, and maintainable web applications**. We'll be
using a wide range of web technologies along with the latest ideas from
emerging languages like Elixir and Elm to craft a fun web experience. Rather
than focusing on theory, we'll take a practical approach and build a real-world
application.

## What We're Building

The application we'll be building will be a small game platform for the web.
We'll use Elixir and the Phoenix web framework to create the back-end, where
our users can log in and keep track of their scores. And then we'll use Elm on
the front-end to create fun minigames. We'll tie everything together so that we
can pass data back and forth between the back-end and front-end. Users on our
platform should be able to join a chat to discuss the different minigames, and
the scores from those games should get synced back to a central database.
**We'll focus on building things with a strong foundation, so that we can use
these same concepts to create different web applications as well**.

## Acknowledgements

I would like to thank Envy Labs and Code School for fostering an environment
where I've been able to work hard and learn and grow. I'd also like to thank
José Valim and Evan Czaplicki for crafting such beautiful and fun languages.
Lastly, thanks to Michael Hartl for setting the standard for technical writing
with The Ruby on Rails Tutorial, and Bret Victor for inspiring all of us with
his visions of the future.

## Who Is This Book For?

This book is written for web developers that already have _some_ existing
experience with web programming. The goal is for the book to be a practical
introduction to building a project with functional web programming languages
like Elixir and Elm.

We won't assume any prior experience with Elixir and Elm, and consider it more
likely that you've worked with languages like Ruby and JavaScript. But **keep
in mind that we'll occasionally forego in-depth explanations and theory in an
effort to gain insight into shipping a real project**.

We'll walk through initial explanations to give you just enough information
about the fundamentals and concepts so you can be productive. But there are
other books that will provide considerably more depth when it comes to learning
the languages themselves:

- [Programming Elixir](https://pragprog.com/book/elixir/programming-elixir)
  by Dave Thomas
- [An Introduction to Elm](https://guide.elm-lang.org/) by Evan Czaplicki

The material in this book is intended to be crafted in such a way that you can
follow along simply by typing in the relevant code examples. So beginners can
still learn a lot simply by following along and building the application,
because sometimes in programming you need to be exposed to certain concepts and
ideas before they become easy to understand. The experience of building
something will be fun and engaging; and a deeper understanding will follow with
increased familiarity and experience.

## Prerequisites

In addition to the notes above about the intended audience for this book, here
are some additional prerequisites to keep in mind:

- Some experience with HTML and CSS.
- Familiarity with the command line and a text editor.
- Preferably previous experience with Git and GitHub.
- Preferably some experience working with a web framework.

## Why Elixir and Elm?

### Elixir

[**Elixir**](http://elixir-lang.org) is a dynamic, **functional** language
designed for building scalable and maintainable applications.

- Elixir is built on top of the Erlang virtual machine, the language inherits
  _decades_ worth of **stability** and **scalability**.
- **Concurrency** is at the heart of Elixir. Instead of getting faster
  processors, computers these days are getting processors with more cores. What
  does that mean? It means that we need to write our programs in such a way that
  allows them to be distributed across multiple cores so that our programs can
  outperform our competitors.

  - For example, you can compare the latest
    [13-inch Macbook Pro models](http://www.apple.com/shop/buy-mac/macbook-pro/13-inch)
    with dual-core processors with
    [15-inch Macbook Pro models](http://www.apple.com/shop/buy-mac/macbook-pro/15-inch)
    with quad-core processors. Then, see how many cores you'll have access to
    when you deploy your application to a
    [multi-core web server](https://www.digitalocean.com/pricing/#droplet).

- The **Phoenix web framework**. For web developers that have worked with Ruby
  on Rails, the concepts will be very familiar and easy to pick up.

- Elixir inherits **amazing features from other languages**.

  - **Ruby**'s readable syntax and philosophy of developer 
  - **Erlang**'s stability and scalability.
  - **F#**'s magical pipeline operator for data transformation.
  - **LISP**'s macros and metaprogramming.

### Elm

[**Elm**](http://elm-lang.org) is an exciting new **functional** language that
is still evolving. It's the fastest, most reliable front-end web programming
language currently available.

- [Presentation](https://prezi.com/wofdk8e6uuy3/getting-to-know-elm)
- Elm is a **compiled, functional** language.
- Elm is **blazingly fast**.
- **No runtime errors**. What does that mean? It means the language was
  designed in such a way that makes a certain class of errors impossible. In
  languages like Ruby and JavaScript, it's common to program in a way that
  incorporates many bugs into our programs.
- The **Elm compiler** can help guide you to writing solid code. **Error
  messages** are extremely helpful.
- The **elm-format** tool can also help for those of us that are new to the
  syntax.
- **Confidence**. Given the benefits mentioned above, you'll have the
  confidence that your code really works.
- **Maintainability**. Refactoring is a dream in languages like Elm, and you'll
  find yourself surprised at how easy a significant refactor can feel after
  coming from other languages.

### Elixir and Elm?

**Elixir and Elm are young, functional programming languages** that are
optimized for **your happiness as a developer**. They offer a programming
experience that will make it fun to develop applications, and down the line
you'll be thankful because your **applications will be easy to extend and
maintain**.

The primary reason to pick up new languages like Elixir and Elm is that it will
afford you with **an opportunity to acquire new ways of thinking**. Many great
lessons have been learned in the field of programming over the past several
decades, and unfortunately many of us are still working in the dark on a daily
basis. **We ignore history at a great cost**, and all too often make things
difficult on ourselves. **Elixir and Elm are a chance at a fresh perspective**.

## Technology Stack

There are _many_ technologies involved in building and deploying a modern web
application. We'll be using a straightforward stack of technologies that will
allow us the flexibility to scale our applications gracefully. Here's the short
version of the technology stack:

- Back-end: Elixir
- Front-end: Elm

These technologies are standing on the shoulders of giants, so here's a little
more information about other technologies we'll also use while building our
applications:

- Back-end: Elixir and Phoenix
- Front-end: Elm and the Elm Architecture and JavaScript
- Version Control: Git and GitHub
- Data: PostgreSQL and JSON
- Deployment: Heroku

## Functional Programming

If you're coming from a background in working with **Ruby on Rails** or
**JavaScript** web frameworks, then you'll have a great head start in being
able to grasp the content and move smoothly through the book. Something to keep
in mind is that **Elixir** and **Elm** are _functional_ languages. If you're
coming from an _object-oriented_ background, you may find some of the
programmatic approaches to be unfamiliar at first, but that initial discomfort
will pay off well in the long run as you learn to solve problems in an elegant
and pleasing manner.

