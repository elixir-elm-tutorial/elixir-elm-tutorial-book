# Introduction

Welcome to the world of functional web programming! In this book, we'll **learn
how to create fun, scalable, and maintainable web applications**. We'll be
using a wide range of web technologies along with the latest ideas from
emerging languages like Elixir and Elm to craft a fun experience. Rather than
focusing on theory, we'll take a practical approach and build a real-world
application.

## What We're Building

**The application we'll be building together is a small game platform for the
web**. We'll use Elixir and the Phoenix web framework to power the back-end,
where players can sign in and keep track of their scores. Then, we'll use Elm
on the front-end to create fun minigames. We'll connect everything together so
we can pass data back and forth between the back-end and front-end. Users on
our platform will view available minigames, and the scores from those games
will be updated on the platform in real-time. We'll focus on building things
with a solid foundation so we can use these same concepts to create different
web applications as well.

## Acknowledgements

I would like to thank **Envy Labs** and **Code School** for fostering
environments where I was able to work hard and learn and grow. I'd also like
to thank **José Valim** and **Evan Czaplicki** for crafting such beautiful and
fun programming languages. And thanks to **Bret Victor** for inspiring all of
us with his visions of the future.

## Who Is This Book For?

This book is written for developers who already have _some_ existing experience
with web programming. The goal is for the book to be a practical introduction
to building a project with functional web programming languages like Elixir and
Elm.

We won't assume any prior experience with Elixir and Elm, and consider it more
likely that you've worked with languages like Ruby and JavaScript. But **keep
in mind that we'll occasionally forego in-depth explanations and theory in an
effort to gain insight into shipping a real project**.

We'll walk through initial explanations to give you just enough information
about the fundamentals and concepts so you can be productive. But there are
other books that will provide more depth when it comes to learning more about
the languages themselves:

- [Programming Elixir](https://pragprog.com/book/elixir13/programming-elixir-1-3)
  by Dave Thomas
- [An Introduction to Elm](https://guide.elm-lang.org) by Evan Czaplicki

The material in this book is intended to be crafted in such a way that you can
follow along simply by typing in the relevant code examples. Beginners can
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

[Elixir](http://elixir-lang.org) is a dynamic, functional language designed for
building scalable and maintainable applications.

- Elixir is built on top of the Erlang virtual machine, and therefore inherits
  _decades_ worth of stability and scalability.
- Concurrency is at the heart of Elixir. Instead of getting faster processors,
  computers these days are getting processors with more cores. That means we
  need to write our programs in such a way that allows them to be distributed
  across multiple cores so our programs can outperform our competitors. As an
  example, compare the latest
  [13-inch Macbook Pro models](http://www.apple.com/shop/buy-mac/macbook-pro/13-inch)
  with dual-core processors with
  [15-inch Macbook Pro models](http://www.apple.com/shop/buy-mac/macbook-pro/15-inch)
  with quad-core processors. Then, see how many cores you'll have access to
  when you deploy your application to a
  [multi-core web server](https://www.digitalocean.com/pricing/#droplet).
- The Phoenix web framework provides us with the ability to create new projects
  quickly. For web developers that have worked with Ruby on Rails, the concepts
  will be familiar and easy to pick up.
- Elixir also inherits amazing features from other languages:

  - Ruby's readable syntax and philosophy of developer happiness.
  - Erlang's stability and scalability.
  - F#'s magical pipe operator for data transformation.
  - LISP's macros and metaprogramming.

### Elm

[Elm](http://elm-lang.org) is an exciting new functional language that is still
evolving. It's the fastest, most reliable front-end web programming language
currently available.

- Elm is a compiled, functional language.
- Elm is _blazingly_ fast.
- Elm programs are free from runtime errors. That means the language was
  designed in such a way that makes a certain class of errors impossible, which
  provides us with an ability to make guarantees about how our programs work.
- The Elm compiler can be a helpful guide towards writing high quality code,
  and the error messages provided are extremely helpful.
- The elm-format tool helps with writing consistent code that is easier to
  read, write, and maintain. While optional, this tool is _highly_ recommended
  for Elm beginners because you can configure it to automatically format code
  when you save a file in your editor.
- With all the features Elm has to offer, the net result is confidence. As
  developers, we can be more confident that our code is performing the way
  we intended, and that our programs will function properly for our users.
- Elm code is maintainable. Refactoring is a dream, and you'll find yourself
  surprised at how easy a significant refactor can feel after coming from other
  languages.

### Elixir and Elm?

**Elixir and Elm are young, functional programming languages that are optimized
for your happiness as a developer. They offer a programming experience that
will make it fun to develop applications, and over time those applications will
be easy to extend and maintain**.

The primary reason to pick up new languages like Elixir and Elm is that it will
afford you with an opportunity to acquire new ways of thinking. Many great
lessons have been learned in the field of programming over the past several
decades, and unfortunately many developers are still working in the dark on a
daily basis. We ignore history at a great cost, and all too often make things
difficult on ourselves. Elixir and Elm are a chance at a fresh perspective.

## Technology Stack

There are _many_ technologies involved in building and deploying modern web
applications. We'll be using a straightforward stack of technologies that will
allow us the flexibility to scale our applications gracefully. Here's the short
version of the technology stack:

- Back-end: Elixir
- Front-end: Elm

These technologies stand on the shoulders of giants, so here's a little more
information about other technologies we'll also use while building our
applications:

- Back-end: Elixir and Phoenix
- Front-end: Elm, the Elm Architecture, and JavaScript
- Version Control: Git and GitHub
- Data: Ecto, PostgreSQL, and JSON
- Deployment: Heroku

## Functional Programming

If you're coming from a background in working with Ruby on Rails or JavaScript
web frameworks, then you'll have a head start in being able to grasp the
content and move smoothly through the book. Something to keep in mind is that
Elixir and Elm are _functional_ languages. If you're coming from an
_object-oriented_ background, you may find some of the programmatic approaches
to be unfamiliar at first, but the initial discomfort will pay off in the long
run as you learn to solve problems in an elegant functional manner.

## Summary

In this introduction, we touched briefly on the application we'll be building
and some of the reasoning for choosing Elixir and Elm as languages. But the
fun part is _creating_ with these technologies and _experiencing_ the benefits
first-hand, so let's dive in and start building our application in the next
chapter.
