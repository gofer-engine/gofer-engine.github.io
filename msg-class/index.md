[🏠 Gofer Engine](https://gofer-engine.github.io/) > Msg Class

# @gofer-engine/hl7

The `@gofer-engine/hl7` is a strongly typed `Msg` TypeScript class to encode, decode, extrapolate, transform, and map over HL7 message data.

This library package was developed as an independent library within `@gofer-engine`. This allows developers to use this package without also needing `@gofer-engine/engine`.

Formerly released as `ts-hl7`, this package was the initial project which spurred the creation of Gofer Engine to bring the simplicity of `ts-hl7` into a complete working Engine and not just a message encapsulation class.

## Purpose

HL7 v2.x has a unique syntax and message structure. Usually, HL7 messages are encoded and decoded into and out of XML structures. While XML structures are flexible enough to handle the unique message structure. They are harder to work with for extrapolating and transforming over the more commonly used JSON data structure. We knew we wanted to store the message data in a format that was compact but could also be easily queried directly and deeply within modern graph and document databases such as MongoDB, SurrealDB, and Dgraph which also work directly with JSON data structures. So for these reasons, we developed this package.

If you are unfamiliar with HL7, the founder Anthony Master has written an article providing a introduction for beginners into the world of HL7: [What is HL7](https://dev.to/amaster507_59/what-is-hl7-4c9f)

## Installation

To use with Node projects, open a terminal/command prompt inside of your node project directory and run

```sh
npm install @gofer-engine/hl7
```

If you want to use this library in a TS/JS project, but outside of a node project, please contact me and we will work out the steps to follow. I have not had a need for this yet.

## Usage

```ts
import Msg from '@gofer-engine/hl7'
// using fs to read sample.hl7 file
import fs from 'fs'

const HL7_String = fs.readFileSync('./sample.hl7', 'utf8')

const msg = new Msg(HL7_String)
```

## Documentation

- [Decoding and Encoding](./encoding-decoding.md)
- [Extrapolating](./extrapolating.md)
- [Transforming](./transforming.md)
- [Sub-Classes](./sub-classes.md)