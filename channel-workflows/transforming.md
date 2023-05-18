## Transform Flows

Transform Flows are used to transform messages. The interface `TransformFlow` can be defined as:

```typescript
type TransformFunc = (msg: Msg, context: IMessageContext) => Msg
type TransformFlow =
  | TransformFunc
  | { kind: 'transform'; transform: TransformFunc }
```

_Refer to the **[Message Class (`Msg`)](../msg-class/index.md)** below for more information on the `Msg` class and transforming the data in the message._

_Refer to the **[Context Object](./context-object.md)** below for more information on the `context` object._

The trasnformer functions of the class retun back the class instance, so you can chain them together. Here is an example of a transformer that takes the field `PV1-3` and adds a prefix to it:

```typescript
const channelConfig: ChannelConfig = {
  name: 'My Channel',
  source: {
    tcp: {
      host: 'localhost',
      port: 8080,
    },
  },
  ingestion: [
    {
      transform: (msg) =>
        msg.map('PV1-3[1].1', (location) => 'HOSP.' + location),
    },
  ],
}
```

This could be refactored a little further to allow for more flexibility:

```typescript
const addPrefix = (path: string, prefix: string) => (msg: Msg) =>
  msg.map(path, (location) => prefix + location)

const channelConfig: ChannelConfig = {
  name: 'My Channel',
  source: {
    tcp: {
      host: 'localhost',
      port: 8080,
    },
  },
  ingestion: [addPrefix('PV1-3[1].1', 'HOSP')],
}
```

For advanced type control, you can pass through a generic to the ChannelConfig (the _second_ generic option) which passes down to the FilterFlow generic to either:

- `'F'` = Only allow raw transform functions. E.G. `ingection: [(msg) => msg]`
- `'O'` = Only allow transform functions in objects. E.G. `ingestion: [{ transform: (msg) => msg }]`
- `'B'` = Allow both raw transform function or wrapped in objects. E.G. `ingestion: [(msg) => msg, { transform: (msg) => msg }]`

The default is `'B'`. E.G. `const conf: ChannelConfig<'B', 'B'> = ...`

The OOP style channel builder `transform` method aligns with the following types:

```ts
IngestionClass.transform = (transform: TransformFlow<'F'>) => IngestionClass
RouteClass.transform = (transform: TransformFlow<'F>) => RouteClass
```

This means that the `transform` method accepts a single argument which is a function.

## Filter or Transform

For flexibility, you can pass through a `TransformOrFilterFlow` to the ingestion array or route flows. This allows you to specify a filter and/or a transformer. The interface `TransformOrFilterFlow` is defined as:

```typescript
type TransformFilterFunction = (
  msg: Msg,
  context: IMessageContext
) => false | Msg
type TransformOrFilterFlow =
  | TransformFilterFunction
  | { kind: 'transformFilter'; transformFilter: TransformFilterFunction }
```

This allows you to write a transformer that can exit early if a condition is not met and return `false` to prevent further processing of the following flows.

The method `filterOrTransform` is not yet implemented for the OOP style channel builder.
