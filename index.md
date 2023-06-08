[🏠 Gofer Engine](./index.md)

# Introducing Gofer Engine

_A HL7 interface engine built for simplicity and speed, strongly typed, and ready to containerize._

```ts
import gofer from 'gofer-engine'

gofer
  .listen('tcp', 'localhost', 5500)
  .ack()
  .store({ file: {} })
  .route(route =>
    route.send('tcp', 'hl7.emr.example.com', 5500)
  )
  .run()
```

This is just a minimal example to setup a pass-through interface engine logging messages to files. Gofer Engins is much more capable than just a simple pass-through engine. It also supports filtering, transforming, queuing, events, routing, and much more.

- Gofer Engine includes strongly typed classes for TypeScript developers. It can run in any Node.JS environment wherever you want to deploy, on a server, on your developing workstation, or in docker containers.
- Easily enable CI/CD pipelines with git repos and develop channels using either OOP or strongly typed configuration files.
- Easily extract, map, set, and append to HL7 messages using our custom built st-hl7 package.

> Reinventing healthcare interoperability tooling!

Best of all, unstainall Java once and for all on your interface servers. Gofer Engine is built around the modern Node.JS stack and does not require any licensed software to run in development nor production. Gofer Engine is open source for transparency and continued community development.

## Documentation

- [Setup, Installation, and Usage](./setup-installation-usage.md)
- [Developing Interface Channels in OOP style](./developing-interface-channels-in-oop.md)
- [Developing Interface Channels with Config Files](./developing-interface-channels-with-configs.md)
- Channel Workflows
  - [Ack Config](./channel-workflows/ack.md)
  - [Filtering](./channel-workflows/filtering.md)
  - [Transforming](./channel-workflows/transforming.md)
  - [Context Object](./channel-workflows/context-object.md)
  - [Queuing](./channel-workflows/queuing.md)
  - [Store Configs](./channel-workflows/store-configs.md)
  - [Routing](./channel-workflows/routing.md)
  - [Events](./channel-workflows/events.md)
  - [Logging](./channel-workflows/logging.md)
- [Working with the HL7 2.x Msg Classes](./msg-class/index.md)
  - [Encoding and Decoding](./msg-class/encoding-decoding.md)
  - [Extrapolating using HL7 Paths](./msg-class/extrapolating.md)
  - [Transforming Messages](./msg-class/transforming.md)
  - [Msg Sub Classes](./msg-class/sub-classes.md)
- [Administrating Gofer Engine](./administrating.md)