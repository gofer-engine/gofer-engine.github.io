[🏠 Gofer Engine](https://gofer-engine.github.io/) > Msg Class

# ts-hl7

`ts-hl7` stands for TypeScript-HL7 and is a strongly typed `Msg` class to encode, decode, extrapolate, transform, and map over HL7 message data.

The `ts-hl7` package was developed as an independent package to gofer-engine. This allows developers to use this package outside of just gofer-engine.

`ts-hl7` was the initial project which spurred the creation of `gofer-engine` to bring the simplicity of `ts-hl7` into a complete working Engine and not just a message encapsulation class.

## Purpose

HL7 v2.x has a unique syntax and message structure. Usually, HL7 messages are encoded and decoded into and out of XML structures. While XML structures are flexible enough to handle the unique message structure. They are harder to work with for extrapolating and transforming over the more commonly used JSON data structure. We knew we wanted to store the message data in a format that was compact but could also be easily queried directly and deeply within modern graph and document databases such as MongoDB, SurrealDB, and Dgraph which also work directly with JSON data structures. So for these reasons, we developed this package.

## Installation

To use with Node projects, open a terminal/command prompt inside of your node project directory and run

```sh
npm install ts-hl7
```

If you want to use `ts-hl7` in a TS/JS project, but outside of a node project, please contact me and we will work out the steps to follow.

## Usage

```ts
import Msg from 'ts-hl7'
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