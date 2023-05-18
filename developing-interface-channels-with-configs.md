# Developing Interface Channels with Configs

Alternatively to the OOP style channel building, you can also use the predated method to configure and run interface channels.

_**NOTE**: Previously the default import `gofer` could be called as a function and passed directly the channel configurations such as `gofer(CHANNELS)`. Since `1.0.0` the default gofer can no longer be called as a function, but is instead a class and the `configs` method provides the backwards compatible functionality like: `gofer.configs(CHANNELS)`.

If you are using TypeScript, the `ChannelConfig` type is the foundation for a Channel Config File. The Channel Config files, are JSON-like structured, but with the added support of adding functions for advance scripting needs and more complex channel configurations.

A simplified view of the `ChannelConfig` type would look like:

```ts
interface ChannelConfig {
  // an optional unique id for this channel.
  // If not provided will use UUID to generate.
  // If not statically defined it may not be the same between deployments/reboots
  id?: string | number
  // a name, preferrably unique, to identify this channel in the logs
  name: string
  // Optional tags to help organize/identify channels
  tags?: Tag[]
  // The source of the messages to process.
  // Currently the only supported source is TCP Listener for HL7 messages.
  source: Connection<'I'>
  // A list of flows to process messages as they are received.
  // The order of the flows is important.
  // Flows will be executed in the order they are defined in this list.
  // If the server should respond to the source, then there should be an ack flow somewhere in this list.
  ingestion: IngestionFlow[]
  // A list of routes composed of flows to process and send messages to other destinations.
  // Each route is a list of flows to process messages as they are received.
  // The order of routes is not important, however the order of the flows in each route is important.
  // If there are asynchronous flows in a route, then other routes can continue to execute while waiting.
  routes?: RouteFlow[][]
  // Optional. If true, will console log additional contextual information
  verbose?: boolean
}
```

## Forcing a Config Style with Generics

The `ChannelConfig` interface can be loosely typed allowing very simple configuration for channels. For example, an `IngestionFlow` can be a function directly:

```typescript
const flow: IngestionFlow = (msg) => msg.get('MSH.9.1') === 'ADT'
```

Or it can be an object with a `kind` property:

```typescript
const flow: IngestionFlow = {
  kind: 'filter',
  filter: (msg) => msg.get('MSH.9.1') === 'ADT',
}
```

The `IngestionFlow` accept generics to force a style. The first generic controlls the Filter flows, and the second generic controlls the Transform flows. The generic is either `'O'` for objects, `'F'` for functions, or `'B'` to allow either (default).

```typescript
const flows: IngestionFlow<'O', 'O'>[] = []
```

To force the use of objects for all filters and transformers in a channel config, you can pass the generics to the `ChannelConfig` interface:

```typescript
const channel: ChannelConfig<'O', 'O'>[] = []
```

More information on the [`ingestionFlow`](#Channel-ingestion) below.

## Channel id

The `id` is optional, If you don't provide an `id`, then the channel will be assigned a UUID which may not be the same between deployments/reboots. The `id` helps to identify ambiguously named channels in the logs.

If you want to force `id`s to be required, then you can pass a third generic to the `ChannelConfig` interface. `'S'` will strictly force the `id` to be required. `'L'` (default) will loosely allow `id` to be undefined.

```typescript
const channel: ChannelConfig<'O', 'O', 'S'>[] = []
```

## Channel name

The `name` is required and should be unique, but not required. The `name` is used to allow human readable channel names in the logs.

## Channel tags

The `tags` are optional and are used to help organize and identify channels. They are not used by the engine, but are there to help you identify related channels and dependencies. The interface `Tag` is currently defined as:

```typescript
interface Tag {
  name: string
  // a hexidecimal color string or valid CSS color name
  color?: string
}
```

Note: These tags are not currently used by the engine or the admin API. Eventually I would like to add a UI to help visualize the channels and their dependencies which would use these tags.

## Channel source

The `source` is required and is the source of the messages to process. Currently the only supported source is TCP Listener for HL7 messages. The interface `Connection<'I'>` is computed to:

```typescript
interface Connection {
  kind: 'tcp'
  tcp: {
    host: string
    port: number
    // Start of Message character. Defaults to '\x0b'
    SoM?: string
    // End of Message character. Defaults to '\x1c'
    EoM?: string
    // End of Transmission character. Defaults to '\r'
    CR?: string
  }
  queue?: QueueConfig
}
```

Future plans include support for HL7 over HTTP, HL7 reading from files in a directory, and HL7 reading from a database. Eventually, I would like to support other message formats such as FHIR, CDA, CSV, PSV, etc.,

More information on the [`QueueConfig`](./channel-workflows/queuing.md) doc page.

## Channel ingestion

The `ingestion` is required and is a list of flows to process messages as they are received. The order of the flows is important. The first flow will be executed first, the second flow will be executed second, etc. If the server should respond to the source, then there should be an ack flow somewhere in this list. The interface `IngestionFlow` is computed to:

```typescript
type IngestionFlow =
  | {
      ack: AckConfig
    }
  | FilterFlow
  | TransformFlow
  | StoreConfig
```

More information on [`FilterFlow`](./channel-workflows/filtering.md) doc page.
More information on [`TransformFlow`](./channel-workflows/transforming.md) doc page.
More information on [`StoreConfig`](./channel-workflows/store-configs.md) doc page.

## Acknowledge Config

```typescript
type AckConfig = {
  // Optional. Value to use in ACK MSH.3 field. Defaults to 'gofer Engine'
  application?: string
  // Optional. Value to use in ACK MSH.4 field. Defaults to ''
  organization?: string
  // Optional. Value to use in MSA-1 field. Default to 'AA'
  responseCode?: 'AA' | 'AE' | 'AR'
  // Optional. A function that accepts the ack MSG class, msg MSG class, and conext state object
  // and returns the ACK MSG class back. This allows for custom transformation of the ACK message.
  msg?: (ack: Msg, msg: Msg, context: IAckContext) => Msg
}
```

More information on [Acknowledgement Workflow](./channel-workflows/ack.md) doc page.
More information on [Msg](./msg-class/index.md)

## Channel routes

The `routes` are optional and are a list of routes composed of flows to process and send messages to other destinations. Each route is a list of flows to process messages as they are received. The order of routes is not important, however the order of the flows in each route is important. If there are asynchronous flows in a route, then other routes can continue to execute while waiting.

The `ChannelConfig` interface was simplified above to show the basic structure. The `routes` property can be lossely defined as multideimensional array of `RouteFlow` or `RouteFlowNamed` interfaces. Or it can be strictly defined as an array of `Route` interfaces typed as:

```typescript
interface Route {
  kind: 'route'
  id?: string | number
  name?: string
  tags?: Tag[]
  queue?: QueueConfig
  flows: RouteFlow[]
}
```

With strict Channel Config (`ChannelConfig<'O', 'O', 'S'>`) then the `routes` property must be defined as a `Route` interface and the `id` becomes required.

Similarly the `Route['flow']` type is simplified above to show the basic structure. The `flows` property can be lossely defined as an array of `RouteFlow` or `RouteFlowNamed` interfaces. Or it can be strictly defined to only include `RouteFlowNamed` interfaces typed as:

```typescript
interface RouteFlowNamed {
  kind: 'flow'
  id?: string | number
  name?: string
  tags?: Tag[]
  queue?: QueueConfig
  flow: RouteFlow
}
```

If you are going to add a queue to a route, then you must use the `Route` interface and not the simplified `RouteFlow[]` array. If you are going to add a queue to a flow, then you must use the `RouteFlowNamed` interface and not the simplified `RouteFlow` interface.

More information on the [`QueueConfig`](./channel-workflows/queuing.md) doc page.

The interface `RouteFlow` is computed to:

```typescript
type RouteFlow =
  | FilterFlow // see (## Filter Flows) below
  | TransformFlow // see (## Transform Flows) below
  | StoreConfig // see (## Store Configs) below
  | {
      kind: 'tcp'
      tcp: {
        host: string
        port: number
        // Start of Message character. Defaults to '\x0b'
        SoM?: string
        // End of Message character. Defaults to '\x1c'
        EoM?: string
        // End of Transmission character. Defaults to '\r'
        CR?: string
        // response timeout in milliseconds. NOTE: not yet implemented
        responseTimeout?: number | false
      }
    }
```

Currently only TCP remote destinations are supported.

More information on the [Routing](./channel-workflows/routing.md) doc page.
