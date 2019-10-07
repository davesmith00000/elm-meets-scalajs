# Elm meets Scala.JS

An example of passing messages between Elm and Scala.js.

I've taken the standard Elm counter example and extended it slightly to demonstrate invoking and responding to Scala.js via messaging.

## Running the example

Assumptions:

- You have checked out the repo
- You have SBT installed
- You have Node & NPM installed
- You have ParcelJS installed
- You have Elm 0.19 installed

From your terminal, move to the `scala` folder and run:

```bash
sbt clean update compile fastOptJS
```

Go back to the root directory.

If this is your first run, install the node dependancies with:

```bash
npm install
```

From then on, you can start the demo off by doing:

```bash
npm start
```

ParcelJS will compile everything (other than the Scala.js asset) and start a service running the example on port `1234`.

Start up a browser window and navigate to `http://localhost:1234` and open your JavaScript console so that you can see the logs.

## Getting started with the code

The scala entry point describes a function for [processing messages](https://github.com/davesmith00000/elm-meets-scalajs/blob/master/scala/src/main/scala/ElmMailbox.scala) of the form `ElmMessage => ScalaMessage`.

The JavaScript bridging code is a [one-liner](https://github.com/davesmith00000/elm-meets-scalajs/blob/master/index.js#L9).

`LogMessage` asks for a message to be [sent to Scala](https://github.com/davesmith00000/elm-meets-scalajs/blob/master/src/Main.elm#L53) to be logged as a side effect.

The `Increment` and `Decrement` message types from the standard example also ask Scala to [double the values](https://github.com/davesmith00000/elm-meets-scalajs/blob/master/src/Main.elm#L44) via a Command, and when [Scala responds with the doubled value](https://github.com/davesmith00000/elm-meets-scalajs/blob/master/src/Main.elm#L56) the model is updated as normal ready for rendering into HTML by the view.

## Motivation

Elm is a great way to do general frontend development. The compile time guarantees, high performance, maintainability and super productive workflow are wonderful to work with. However the strict and holistic design philosophy that enables all those wonderful qualities in Elm, does produce pain points too.

This toy project aims to answer one of the pain points:
Since Elm has no way to natively call JavaScript, there are only two ways to extend the standard Elm functionality.

1. Elm's packages, which are often great but occassionally lacking in some key areas;
2. Calling down to plain old JavaScript via Elm's ports system.

The aim for this solution was to end up with a complete FP experience, with a typechecked pattern match of messages on both the Elm and Scala.js sides of the divide, for strong well typed communications.

This would allow you to do all your general web dev in Elm, then drop down to Scala.js when you needed more power and flexibility, playing to the strengths of both languages.

## Scala.js

This approach would work equally well in any language that can be compiled to JavaScript.

That said, for the work we're going to be doing it would seem to make sense to use an impure statically typed functional language, with a Foreign Function Interface (FFI - ability to call native JS), excellent functional abstractions, great library support, and a good compiler.

Scala.js fits the bill perfectly!

## Elm and JavaScript

Elm is complied to JavaScript, but does not permit you to call native JavaScript functions.

This makes sense since in respect to Elm's purpose, which is to offer compile time correctness guarantees and highly maintainable code.

In other general purpose compile-to-JS languages, you interface with JavaScript by creating a type definition file / facade that describes every function in your favourite JS library. You end up doing normal JavaScript development with a bit more type / compiler support, but your code remains tightly coupled with the JavaScript libraries you're using.

In contract, the intention of Elm ports is that you asynchronously call down to JavaScript and ask for a whole blocks of well encapsulated business logic to be invoked. This promotes strong decoupling of work being done in Elm versus the work being done on the otherside of the port. Elm can then enforce type correctness on messages going out or coming back in, making it clear on which side of the port any errors lie.

## Performance considerations

Given that the solution presented is both asynchronous and relys on JSON encoding (Elm), decoding (Ccala), encoding (Scala), and finally decoding (Elm), it is probably slow and inefficient (no benchmarking has been done).

Important to note that although it is not expected to be a particularly performant solution, the anticipated invokation frequency is in response to *human* activity i.e. as far as a computer is concerned, very infrequent indeed. That being the case, performance is actually not a concern at all.

Equally, doing this many times in something like a game loop would probably be a terrible idea!
