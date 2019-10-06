# Elm meets Scala.JS

An example of two-way messaging between Elm and Scala.js.

I've taken the standard Elm "counter" example and extended it slightly to demonstrate invoking and responding to Scala.js via messaging.

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

`LogMessage` asks for a message to be [sent to Scala](https://github.com/davesmith00000/elm-meets-scalajs/blob/master/src/Main.elm#L53) to be logged as a side effect.*

The `Increment` and `Decrement` message types from the standard example also ask Scala to [double the values](https://github.com/davesmith00000/elm-meets-scalajs/blob/master/src/Main.elm#L44) via a Command, and when [Scala responds with the doubled value](https://github.com/davesmith00000/elm-meets-scalajs/blob/master/src/Main.elm#L56) the model is updated as normal ready for rendering into HTML by the view.

(* Needs more thought. Currently even a side effecting action returns a message to Elm, it's just that the event is a NoOp in Elm. That's probably fine since Elm is very efficient. On the other hand: Should the JS bridge filter out NoOps? Should the Scala function allow you to optionally return a message? Is there some better encoding for saying "no need to tell Elm about this..."? A problem to solve another day.)

## Motivation

Elm is a great way to do general frontend development. The compile time guarantees, high performance, maintainability and super productive workflow are wonderful to work with. However, the strict and holistic design philosophy that enables all those wonderful qualities does produce pain points too.

One of the pain points this toy project aims to answer, is that since Elm has no way to natively call JavaScript, there are only two ways to extend the standard Elm experience. The first is through Elm's packages, often more than sufficent, but occassionally lacking in bothersome key areas. The second is to call down to JavaScript via Elm's ports system, but that leaves you in the world of JavaScript when, if you're like me, you were probably hoping for a statically typed functional frontend programming experience.

The aim for this solution was, essentially, to end up with a type checked pattern match of a Union / ADT on both the Elm and Scala sides of the divide, for strong, well typed communications.

This would allow you to do all your general web dev in Elm, then drop down to Scala.js when you needed more power and flexibility, playing to the strengths of both languages.

## Scala.js

This approach would work equally well in any language that can be compiled to JavaScript.

That said, for the work we're going to be doing it probably makes sense to use an impure statically typed functional language that supports a Foreign Function Interface, pattern matching, excellent functional abstractions, great library support, and a good compiler.

Scala.js fits the bill perfectly!

## Elm and JavaScript

Elm is complied to JavaScript, but does not permit you to call native JavaScript functions. This makes sense since Elm's purpose is to offer compile time correctness guarantees, and if you could just call any old JavaScript code there would be no way to enforce those guarantees in a general way.

The idea in Elm is **NOT** to create a type definition file / facade encapsulating every function in your favourite JS library - as you might in TypeScript or Scala.js - and then call it function by function.

The intention of Elm ports is that you asynchronously call down to JavaScript and ask for a whole blocks of well encapsulated business logic to be invoked. This promotes strong decoupling of work being done in Elm versus the work being done on the otherside of the port.

Elm can then enforce type correctness on messages going out or coming back in, making it clear on which side of the port any errors lie.

## Performance considerations

Given that the solution presented is both asynchronous and relys on JSON encoding (Elm), decoding (Ccala), encoding (Scala), and finally decoding (Elm), it is probably slow and inefficient (no benchmarking has been done).

Important to note that although it is not expected to be a particularly performant solution, the anticipated invokation frequency is in response to *human* activity i.e. as far as a computer is concerned, very infrequent indeed. That being the case, performance is actually not a concern at all.

Equally, doing this many times in something like a game loop would probably be a terrible idea!
