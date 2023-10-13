[🏠 Gofer Engine](https://gofer-engine.github.io/) > [Channel Workflows](./index.md) > Routing

# Routing

"Routing" is the term used for outbound processing of messages in the Gofer Engine. Developing a channel route(s), you can build a route with either the JSON config route following the `Route` type, or you can use the OOP style and follow the `ORoute` interface.

- [Developing Routes using JSON Style](#developing-routes-using-json-style)
- [Developing Routes With OOP Style](#developing-routes-with-oop-style)

Simplified types:

```ts
type Route = {
  kind: "route";
  id?: string | number; // a unique id for this route flow. If not provided will use UUID to generate. if not defined it may not be the same between deployments/reboots
  name?: string; // a human readable name for this route flow. Preferrably unique
  tags?: Tag[]; // Tags to help organize/identify route flows
  queue?: QueueConfig;
  flows:
    | RequiredProperties<RouteFlowNamed, "id">[]
    | (RouteFlow | RouteFlowNamed)[];
};

interface ORoute {
  name: (name: string) => ORoute;
  id: (id: string | number) => ORoute;
  filter: (f: FilterFlow) => ORoute;
  transform: (t: TransformFlow) => ORoute;
  store: (s: StoreConfig) => ORoute;
  setMsgVar: (varName: string, varValue: MsgVar) => ORoute;
  setChannelVar: <V>(varName: string, varValue: MsgVar<V>) => ORoute;
  setGlobalVar: <V>(varName: string, varValue: MsgVar<V>) => ORoute;
  getMsgVar: <V>(varName: string, getVal: WithVarDo<V>) => ORoute;
  getChannelVar: <V>(varName: string, getVal: WithVarDo<V>) => ORoute;
  getGlobalVar: <V>(varName: string, getVal: WithVarDo<V>) => ORoute;
  setVar: <V>(scope: varTypes, varName: string, varValue: MsgVar<V>) => ORoute;
  getVar: <V>(scope: varTypes, varName: string, getVal: WithVarDo<V>) => ORoute;
  setRouteVar: <V>(varName: string, varValue: MsgVar<V>) => ORoute;
  getRouteVar: <V>(varName: string, getVal: WithVarDo<V>) => ORoute;
  send: (method: "tcp", host: string, port: number) => ORoute;
  export: () => RequiredProperties<Route, "id" | "flows">;
}

export type RouteFlow =
  | FilterFlow
  | TransformFlow
  | TransformOrFilterFlow
  | ({ kind: "store" } & StoreConfig)
  | Connection;

type RouteFlowNamed = {
  kind: "flow";
  id?: string | number; // a unique id for this route flow. If not provided will use UUID to generate. if not defined it may not be the same between deployments/reboots
  name?: string; // a human readable name for this route flow. Preferrably unique
  tags?: Tag[]; // Tags to help organize/identify route flows
  queue?: QueueConfig;
  flow: RouteFlow;
};

type varTypes = "Global" | "Channel" | "Route" | "Msg";

type MsgVar = any | ((msg: IMsg, context?: IMessageContext) => any);

type WithVarDo = (
  v: any,
  msg: IMsg,
  context: IMessageContext
) => void | IMsg | boolean;
```

_Refer to the **[Message Class (`Msg`)](../msg-class/index.md)** below for more information on the `IMsg` interface and transforming the data in the message._

_Refer to the **[Context Object](./context-object.md)** below for more information on the `context` type._

_Refer to the **[Filter Flows](./transforming.md)** below for more information on the `FilterFlow` type._

_Refer to the **[Transform Flows](./transforming.md)** below for more information on the `TransformFlow` type._

_Refer to the **[Store Configs](./store-configs.md)** below for more information on the `StoreConfig` type._

_Refer to the **[Queuing](./queuing.md)** below for more information on the `QueueConfig` type._

## Developing Routes using JSON Style

Using object configs offer simplified strongly typed configuration and allows the configs to be easily stringified to JSON for exporting/importing

```typescript
import { Route, ChannelConfig } from '@gofer-engine/engine'

const sendMsg: Route = {
  kind: 'route', // required for strong typing
  id: 'ID_1',
  name: 'Example Route',
  tags: [{ name: 'Examples' }],
  queue: {},
  flows: [
    {
      kind: 'flow',
      id: 'ID_2',
      name: 'Example Flow',
      tags: [{ name: 'Examples' }],
      queue: {},
      flow: {
        tcp: {
          host: '192.168.1.200',
          port: 5600
        },
      },
    },
  ],
}

const storeACK: Route = {
  kind: 'route', // required for strong typing
  id: 'ID_3',
  name: 'Example ACK Store Route',
  tags: [{ name: 'Examples' }],
  queue: {},
  flows: [
    {
      kind: 'flow',
      id: 'ID_4',
      name: 'Example ACK Store Flow',
      tags: [{ name: 'Examples' }],
      queue: {},
      flow: {
        store: {
          file: {},
        },
      },
    },
  ],
}

const channel: ChannelConfig = {
  name: 'Example Channel',
  source: {

  },
  ingestion: [{
    kind: 'ack',
    ask: { organization: 'Example Org' }
  }],
  routes: [[sendMsg, storeACK]]
}
```

The `routes` property in the `ChannelConfig` type takes a multidimensional array of `Route`s. This allows routes to be processed syncronously or asyncronously. Routes within the same inner array will be called sequentially. If a route sends to a tcp server, then the next route will be using the ACK returned by that server.

## Developing Routes with OOP Style

- [name](#name)
- [id](#id)
- [filter](#filter)
- [transform](#transform)
- [store](#store)
- [setVar](#setVar)
- [setMsgVar](#setMsgVar)
- [setRouteVar](#setRouteVar)
- [setChannelVar](#setChannelVar)
- [setGlobalVar](#setGlobalVar)
- [getVar](#getVar)
- [getMsgVar](#getMsgVar)
- [getRouteVar](#getRouteVar)
- [getChannelVar](#getChannelVar)
- [getGlobalVar](#getGlobalVar)
- [send](#send)
- [export](#export)


## name

Call the `name` method to name the route 

```typescript
// example.ts (continued)
route.name('A Unique Channel Name')
```

_[Back to top](#developing-routes-with-oop-style)_

## id

Call the `id` method to override the generated id given to the route. If not provided the id will be a UUID

```typescript
// example.ts (continued)
route.id(42)
```

_[Back to top](#developing-routes-with-oop-style)_

## filter

Call the `filter` method to filter the message. See [Filter Flow](./filtering.md) for the function definition

```typescript
// example.ts (continued)
route.filter((msg) => msg.get('MSH-9.1') === "ADT")

// alternatively you can use the long form get classes
route.filter((msg) => 
  msg
    .getSegment('MSH')
    .getField(9)
    .getComponent(1)
    .toString() === 'ADT'
)
```

_[Back to top](#developing-routes-with-oop-style)_

## transform

Call the `transform` method to transform/modify the message. See [Transform Flow](./transforming.md) for the function definition

```typescript
// example.ts (continued)
route.transform((msg) => msg.set('MSH-5'), 'Gofer')
```

_[Back to top](#developing-routes-with-oop-style)_

## store

Call the `store` method to persist the message to a data store. See [Store Config](./store-configs.md) for the config definition

```typescript
// example.ts (continued)
route.store({ file: {} })
```

_[Back to top](#developing-routes-with-oop-style)_

## setVar

Call the `setVar` method to set a variable value for the specific scope. The `varValue` can either be the value itself or a function to callback to retrieve the value from the message and context. See [Variables](./variables.md) for more information on using variables.

```typescript
// example.ts (continued)
route.setVar('Msg', 'name', 'FirstName')

// you can extract the value from the message
route.setVar('Channel', 'facility', (msg) => msg.get('MSH-3.1'))

// you can strictly specify the type of the value
route.setVar<{ bar: string }>('Global', 'foo', { bar: 'baz' })
```

_[Back to top](#developing-routes-with-oop-style)_

## setMsgVar

Call the `setMsgVar` method to set a variable value for the Message scope. Later in this message will be able to use this variable. The `varValue` can either be the value itself or a function to callback to retrieve the value from the message and context. See [Variables](./variables.md) for more information on using variables.

```typescript
// example.ts (continued)
route.setMsgVar('name', 'FirstName')
```

_[Back to top](#developing-routes-with-oop-style)_

## setRouteVar

Call the `setMsgVar` method to set a variable value for the Message scope. Later in this message will be able to use this variable. The `varValue` can either be the value itself or a function to callback to retrieve the value from the message and context. See [Variables](./variables.md) for more information on using variables.

```typescript
// example.ts (continued)
route.setMsgVar('name', 'FirstName')
```

_[Back to top](#developing-routes-with-oop-style)_

## setChannelVar

Call the `setChannelVar` method to set a variable value for the Channel scope. Later in this message or following messages within this same channel will be able to use this variable. The `varValue` can either be the value itself or a function to callback to retrieve the value from the message and context. See [Variables](./variables.md) for more information on using variables.

```typescript
// example.ts (continued)
route.setChannelVar('facility', msg => msg.get('MSH-3.1'))
```

_[Back to top](#developing-routes-with-oop-style)_

## setGlobalVar

Call the `setGlobalVar` method to set a variable value for the Global scope. Anywhere later in current or following messages within this same server will be able to use this variable. The `varValue` can either be the value itself or a function to callback to retrieve the value from the message and context. See [Variables](./variables.md) for more information on using variables.

```typescript
// example.ts (continued)
route.setGlobalVar('foo', { bar: 'baz' })
```

_[Back to top](#developing-routes-with-oop-style)_

## getVar

Call the `getVar` method to get a previously set variable for the given scope by name. Define the callback function (`cb`) to do something with the value of the variable. You can use the value to filter or transform the message, or do something with the [`MessageContext`](./channel-workflows/context-object.md).

- To filter the message, return a boolean.
- To transform the message, return the transformed Msg class instance
- You can return undefined or even not return anything (`void`)

```typescript
// example.ts (continued)
route.getVar('Msg', 'name', (name) => console.log(name))
```

_[Back to top](#developing-routes-with-oop-style)_

## getMsgVar

Call the `getMsgVar` method to get a previously set variable for the Msg scope.

For example, you can use the value to transform the message

```typescript
// example.ts (continued)
route.getMsgVar<string>('name', (name, msg) => msg.set('PID-5.2', name))
```

_[Back to top](#developing-routes-with-oop-style)_

## getRouteVar

Call the `getRouteVar` method to get a previously set variable for the Route scope.

For example, you can use the value to transform the message

```typescript
// example.ts (continued)
route.getRouteVar<string>('name', (name, msg) => msg.set('PID-5.2', name))
```

_[Back to top](#developing-routes-with-oop-style)_


## getChannelVar

Call the `getChannelVar` method to get a previously set variable for the Channel scope.

For example, you can use the value and the context to log a message

```typescript
// example.ts (continued)
route.getChannelVar<string>('facility', (facility, _, { logger }) => {
  logger(`Received message from ${facility}`, 'info')
})
```

_[Back to top](#developing-routes-with-oop-style)_

## getGlobalVar

Call the `getGlobalVar` method to get a previously set variable for the Channel scope

For example, you can use the value and filter the message by returning false

```typescript
// example.ts (continued)
route.getGlobalVar<{ bar: string }>('foo', ({ bar }) => bar === 'foo')
```

_[Back to top](#developing-routes-with-oop-style)_

## send

Call the `send` method to configure a destination to send the message.

Currently only TCP (MLLP) destinations are supported, but coming soon, Gofer Engine will also support ftp, sftp, http, https, and database direct queries/mutations.

For example, you can send the message to an IP and port provided by your EHR

```typescript
// example.ts (continue)
route.send({ tcp: { host: '192.168.1.200', port: 5700 } })
```

You can also use a function to extract this information from the message or context

```typescript
// example.ts (continue)
route.send({ tcp: {
  host: (_msg, context) => context.getRouteVar<string>('host'),
  port: (_msg, context) => context.getRouteVar<number>('port')
} })
```

_[Back to top](#developing-routes-with-oop-style)_

## export

Call the `export` method to save the configuration to a JSON object. This method is mainly used for testing purposes in Gofer Engine.

_[Back to top](#developing-routes-with-oop-style)_

# Advanced Config Style Conformance

For advanced type control, you can pass through generics to the `Route`, `RouteFlow`, and `RouteFlowNamed` typed.

1. The first generic seen in source as `Filt` controls the type of the filter style to either (`F`) Functional configs, (`O`) Ojectified configs, or (`B`) to allow Both config styles.
2. The second generic seen in source as `Tran` controls the type of the transformer style to either (`F`) Functional configs, (`O`) Ojectified configs, or (`B`) to allow Both config styles.
3. The third generic seen in source as `Stct` controls controls the strictness of the configs to either (`S`) Strictly require objectified configs with `id`s, or (`L`) then allow looser config without requirind `id`s.
