[🏠 Gofer Engine](https://gofer-engine.github.io/) > [Channel Workflows](./index.md) > Filtering

## Filter Flows

Filter Flows are used to filter messages. They are used to determine if a message should be processed further or if it should be dropped. The interface `FilterFlow` can be defined as:

```typescript
type FilterFunc = (msg: Msg, context: IMessageContext) => boolean
type FilterFlow = FilterFunc | { kind: 'filter'; filter: FilterFunc }
```

_Refer to the **[Message Class (`Msg`)](../msg-class/index.md)** for more information on the `Msg` class and extrapulating data from the message to use in comparisons._

_Refer to the **[Context Object](./context-object.md)** for more information on the `context` object._

If the filter function returns `true`, then the message will be processed further. If the filter functions returns `false`, then the message will be dropped. An easy cathy phrase to remember is "If it's true, then let it through. If it's false, then it will halt."

Here is a simple example of a filter that will only allow ADT event messages to be processed further:

```typescript
const filter = (msg: Msg) => msg.get('MSH-9.1') === 'ADT'

// OOP style
gofer.listen('tcp', 'localhost', 8080)
  .filter(filter)

// Config style
const channelConfig: ChannelConfig = {
  name: 'ADT Channel',
  source: {
    tcp: {
      host: 'localhost',
      port: 8080,
    },
  },
  ingestion: [{ filter }],
}
```

This could be refactored a little further to allow for more flexibility:

```typescript
const onlyAllowEvents = (event: string[]) => (msg: Msg) =>
  event.includes(msg.get('MSH-9.1') as string)

// OOP style
gofer.listen('tcp', 'localhost', 8080)
  .filter(onlyAllowEvents(['ADT', 'ORM', 'ORU']))

// Config style
const channelConfig: ChannelConfig = {
  name: 'ADT/ORM/ORU Channel',
  source: {
    tcp: {
      host: 'localhost',
      port: 8080,
    },
  },
  ingestion: [onlyAllowEvents(['ADT', 'ORM', 'ORU'])],
}
```

For advanced type control, you can pass through a generic to the ChannelConfig (the _first_ generic option) to either:

- `'F'` = Only allow raw filter functions. E.G. `ingestion: [() => true]`
- `'O'` = Only allow filter functions in objects. E.G. `ingestion: [{ filter: () => true }]`
- `'B'` = Allow both raw filter function or wrapped in objects. E.G. `ingestion: [() => true, { filter: () => true }]`

The default is `'B'`. E.G. `const conf: ChannelConfig<'B'> = ...`

The OOP style channel builder `filter` method aligns with the following types:

```ts
IngestionClass.filter = (filter: FilterFlow<'F'>) => IngestionClass
RouteClass.filter = (filter: FilterFlow<'F>) => RouteClass
```

This means that the `filter` method accepts a single argument which is a function.
