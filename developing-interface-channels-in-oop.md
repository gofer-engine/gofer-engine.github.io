[🏠 Gofer Engine](./index.md) > Developing Interface Channels in OOP Style

# Developing Interface Channels in OOP Style

We have strived to make developing interface channels as easy as possible. The goal is to make it so that you can create a channel in a few minutes and have it running immediately in a test environment and then deploy it to a production environment with minimal effort.
The Gofer Engine class has methods to quickly and easily create and run channel configuration scripts. The default export is already an instance of the class, so you can just start using it immediately.

- [listen](#listen)
- [IngestionClass](#ingestionclass)
  - [`name`](#name)
  - [`id`](#id)
  - [`ack`](#ack)
  - [`filter`](#filter)
  - [`transform`](#transform)
  - [`store`](#store)
  - [`setVar`](#setvar)
  - [`setMsgVar`](#setmsgvar)
  - [`setChannelVar`](#setchannelvar)
  - [`setGlobalVar`](#setglobalvar)
  - [`getVar`](#getvar)
  - [`getMsgVar`](#getmsgvar)
  - [`getChannelVar`](#getchannelvar)
  - [`getGlobalVar`](#getglobalvar)
  - [`route`](#route)
  - [`routes`](#routes)
- [RouteClass](#routeclass)
  - `name` see [`IngestionClass.name`](#name)
  - `id` see [`IngestionClass.id`](#id)
  - `filter` see [`IngestionClass.filter`](#filter)
  - `transform` see [`IngestionClass.transform`](#transform)
  - `store` see [`IngestionClass.store`](#store)
  - `setVar` see [`IngestionClass.setVar`](#setvar)
  - `setMsgVar` see [`IngestionClass.setMsgVar`](#setmsgvar)
  - [`setRouteVar`](#setroutevar)
  - `setChannelVar` see [`IngestionClass.setChannelVar`](#setchannelvar)
  - `setGlobalVar` see [`IngestionClass.setGlobalVar`](#setglobalvar)
  - `getVar` see [`IngestionClass.getVar`](#getvar)
  - `getMsgVar` see [`IngestionClass.getMsgVar`](#getmsgvar)
  - [`getRouteVar`](#setroutevar)
  - `getChannelVar` see [`IngestionClass.getChannelVar`](#getchannelvar)
  - `getGlobalVar` see [`IngestionClass.getGlobalVar`](#getglobalvar)
  - [`send`](#send)
- [CompleteClass](#completeclass)
  - [`run`](#run)
  - [`export`](#export)
  - (FUTURE) [`msg`](#msg)

## listen

To begin a new channel, use the `listen` method. This method accepts three arguments for TCP listeners:

- `method`: `"tcp"`
- `host`: A `string` for the ip or hostname on which to listen
- `port`: The `number` of the port to listen on.

Or two arguments for HTTP and HTTPS:

- `method`: `"http"`
- `options`: `HTTPConfig<'I'>`

- `method`: `"https"`
- `options`: `HTTPSConfig<'I'>`

See [source](./developing-interface-channels-with-configs.md#source) for typings of `HTTPConfig<'I'>` and `HTPPSConfig<'I'>`

_**Note**: Currently only TCP, HTTP, and HTTPS listeners are supported. In a future release, there will be additional methods to listen, read, and accept messages._

```typescript
// example.ts with TCP listener
import gofer,  { ChannelConfig } from '@gofer-engine/engine'

const ingest = gofer.listen('tcp', 'localhost', 5501)
// we will continue building upon this snippet
```

```typescript
// example.ts with HTTP listener
import gofer,  { ChannelConfig } from '@gofer-engine/engine'

const ingest = gofer.listen('http', {
  host: '127.0.0.1',
  port: 8100,
  method: 'POST',
  basicAuth: {
    username: 'user',
    password: 'pass',
  },
})
// we will continue building upon this snippet
```

```typescript
// example.ts with HTTPS listener
import gofer,  { ChannelConfig } from '@gofer-engine/engine'
import fs from 'fs'

// could use .env variables instead
const SSL_KEY = fs.readFileSync('yourpath/key.pem')
const SSL_CERT = fs.readFileSync('yourpath/cert.pem')

const ingest = gofer.listen('tcp', {
  host: '127.0.0.1',
  port: 8200,
  method: 'POST',
  basicAuth: {
    username: 'user',
    password: 'pass',
  },
  key: SSL_KEY,
  cert: SSL_CERT,
})
// we will continue building upon this snippet
```



The `listen` method returns an [`IngestionClass`](#ingestionclass).

_[Back to top](#developing-interface-channels-in-oop-style)_

## IngestionClass

The `IngestionClass` provides the following methods:

### name

Call the `name` method to name the channel 

```typescript
// definition
type name = (name: string) => IngestionClass
```

```typescript
// example.ts (continued)
ingest.name('A Unique Channel Name')
```

_[Back to top](#developing-interface-channels-in-oop-style)_

### id

Call the `id` method to override the generated id given to the channel. If not provided the id will be a UUID

```typescript
// definition
type id = (id: string | number) => IngestionClass
```

```typescript
// example.ts (continued)
ingest.id(42)
```

_[Back to top](#developing-interface-channels-in-oop-style)_

### ack

Call the `ack` method to reply back an HL7 acknowledgment. See [Acknowledge Config](./channel-workflows/ack.md)

```typescript
// definition
type ack = (ack?: AckConfig) => IngestionClass
```

```typescript
// example.ts (continued)
ingest.ack()
```

_[Back to top](#developing-interface-channels-in-oop-style)_

### filter

Call the `filter` method to filter the message. See [Filter Flow](./channel-workflows/filtering.md) for the function definition

```typescript
// definition
type filter = (filter: FilterFlow<'F'>) => IngestionClass
```

```typescript
// example.ts (continued)
ingest.filter((msg) => msg.get('MSH-9.1') === "ADT")

// alternatively you can use the long form get classes
ingest.filter((msg) => 
  msg
    .getSegment('MSH')
    .getField(9)
    .getComponent(1)
    .toString() === 'ADT'
)
```

_[Back to top](#developing-interface-channels-in-oop-style)_

### transform

Call the `transform` method to transform/modify the message. See [Transform Flow](./channel-workflows/transforming.md) for the function definition

```typescript
// definition
type transform = (transform: TransformFlow<'F'>) = > IngestionClass
```

```typescript
// example.ts (continued)
ingest.transform((msg) => msg.set('MSH-5'), 'Gofer')
```

_[Back to top](#developing-interface-channels-in-oop-style)_

### store

Call the `store` method to persist the message to a data store. See [Store Config](./channel-workflows/store-configs.md) for the config definition

```typescript
// definition
type store = (store: StoreConfig) => IngestionClass
```

```typescript
// example.ts (continued)
ingest.store({ file: {} })
```

_[Back to top](#developing-interface-channels-in-oop-style)_

### setVar

Call the `setVar` method to set a variable value for the specific scope. The `varValue` can either be the value itself or a function to callback to retrieve the value from the message and context. See [Variables](./channel-workflows/variables.md) for more information on using variables.

```typescript
// definition
type setVar = <V>(scope: 'Msg' | 'Channel' | 'Global', varName: string, varValue: V | (msg: Msg, context?: IMessageContext) => V) => IngestionClass
```

```typescript
// example.ts (continued)
ingest.setVar('Msg', 'name', 'FirstName')

// you can extract the value from the message
ingest.setVar('Channel', 'facility', (msg) => msg.get('MSH-3.1'))

// you can strictly specify the type of the value
ingest.setVar<{ bar: string }>('Global', 'foo', { bar: 'baz' })
```

_[Back to top](#developing-interface-channels-in-oop-style)_

### setMsgVar

Call the `setMsgVar` method to set a variable value for the Message scope. Later in this message will be able to use this variable. The `varValue` can either be the value itself or a function to callback to retrieve the value from the message and context. See [Variables](./channel-workflows/variables.md) for more information on using variables.

```typescript
// definition
type setMsgVar = <V>(varName: string, varValue: V | (msg: Msg, context?: IMessageContext) => V) => IngestionClass
```

```typescript
// example.ts (continued)
ingest.setMsgVar('name', 'FirstName')
```

_[Back to top](#developing-interface-channels-in-oop-style)_

### setChannelVar

Call the `setChannelVar` method to set a variable value for the Channel scope. Later in this message or following messages within this same channel will be able to use this variable. The `varValue` can either be the value itself or a function to callback to retrieve the value from the message and context. See [Variables](./channel-workflows/variables.md) for more information on using variables.

```typescript
// definition
type setChannelVar = <V>(varName: string, varValue: V | (msg: Msg, context?: IMessageContext) => V) => IngestionClass
```

```typescript
// example.ts (continued)
ingest.setChannelVar('facility', msg => msg.get('MSH-3.1'))
```

_[Back to top](#developing-interface-channels-in-oop-style)_

### setGlobalVar

Call the `setGlobalVar` method to set a variable value for the Global scope. Anywhere later in current or following messages within this same server will be able to use this variable. The `varValue` can either be the value itself or a function to callback to retrieve the value from the message and context. See [Variables](./channel-workflows/variables.md) for more information on using variables.

```typescript
// definition
type setGlobalVar = <V>(varName: string, varValue: V | (msg: Msg, context?: IMessageContext) => V) => IngestionClass
```

```typescript
// example.ts (continued)
ingest.setGlobalVar('foo', { bar: 'baz' })
```

_[Back to top](#developing-interface-channels-in-oop-style)_

### getVar

Call the `getVar` method to get a previously set variable for the given scope by name. Define the callback function (`cb`) to do something with the value of the variable. You can use the value to filter or transform the message, or do something with the [`MessageContext`](./channel-workflows/context-object.md).

- To filter the message, return a boolean.
- To transform the message, return the transformed Msg class instance
- You can return undefined or even not return anything (`void`)

```typescript
// definition
type getVar = <V>(scope: 'Msg' | 'Channel' | 'Global', varName: string, cb: (val: V | undefined, msg: Msg, context: IMessageContext) => void | Msg | boolean) => IngestionClass
```

```typescript
// example.ts (continued)
ingest.getVar('Msg', 'name', (name) => console.log(name))
```

_[Back to top](#developing-interface-channels-in-oop-style)_

### getMsgVar

Call the `getMsgVar` method to get a previously set variable for the Msg scope.

For example, you can use the value to transform the message

```typescript
// definition
type getMsgVar = <V>(varName: string, cb: (val: V | undefined, msg: Msg, context: IMessageContext) => void | Msg | boolean) => IngestionClass
```

```typescript
// example.ts (continued)
ingest.getMsgVar<string>('name', (name, msg) => msg.set('PID-5.2', name))
```

_[Back to top](#developing-interface-channels-in-oop-style)_

### getChannelVar

Call the `getChannelVar` method to get a previously set variable for the Channel scope.

For example, you can use the value and the context to log a message

```typescript
// definition
type getChannelVar = <V>(varName: string, cb: (val: V | undefined, msg: Msg, context: IMessageContext) => void | Msg | boolean) => IngestionClass
```

```typescript
// example.ts (continued)
ingest.getChannelVar<string>('facility', (facility, _, { logger }) => {
  logger(`Received message from ${facility}`, 'info')
})
```

_[Back to top](#developing-interface-channels-in-oop-style)_

### getGlobalVar

Call the `getGlobalVar` method to get a previously set variable for the Channel scope

For example, you can use the value and filter the message by returning false

```typescript
// definition
type getGlobalVar = <V>(varName: string, cb: (val: V | undefined, msg: Msg, context: IMessageContext) => void | Msg | boolean) => IngestionClass
```

```typescript
// example.ts (continued)
ingest.getGlobalVar<{ bar: string }>('foo', ({ bar }) => bar === 'foo')
```

_[Back to top](#developing-interface-channels-in-oop-style)_

### route

Call the `route` method to route the message to a _single_ destination. This method returns an instance of [`CompleteClass`](#completeclass).

The argument to the route is a function that provides an instance of `RouteClass` and must return an instance of `RouteClass`. See [`RouteClass`](#routeclass) for the available methods.

```typescript
// definition
type route = (route: (route: RouteClass) => RouteClass) => CompleteClass
```

```typescript
// example.ts (continued)
const comp = ingest.route(route => route.send('tcp', 'localhost', 5502))
```

_[Back to top](#developing-interface-channels-in-oop-style)_

### routes

Call the `routes` method to route the message to _multiple_ routes. This method returns an instance of [`CompleteClass`](#completeclass).

The argument to the routes is a function that provides a getter of an instance of a `RouteClass` and must return an array of instances of `RouteClasses`. This allows multiple routes to be initiated from a single-getter function. See [`RouteClass`](#routeclass) for the available methods.

Notice that each array begins by calling the `route` function. This is different from the single route method above.

```typescript
// definition
type routes = (routes: (route: () => RouteClass) => RouteClass[]) => CompleteClass
```

```typescript
// example.ts (continued)
const comp = ingest.routes(route => [
  // first route
  route().send('tcp', 'localhost', 5502),
  // second route
  route().transform(msg => msg.addSegment('ZZZ|1|GoferEngine')).send('tcp', 'localhost', 5503),
  // third route
  route().getChannelVar<string>('facility', (facility) => facility === 'FACILITY_3').send('tcp', 'localhost', 5504),
])
```

_[Back to top](#developing-interface-channels-in-oop-style)_

## RouteClass

The RouteClass has all of the above methods of the [`IngestionClass`](#ingestionclass) excluding `ack`, `route`, and `routes`. The `setVar` and `getVar` have the additional scope `"Route"`, and the RouteClass has these additional methods:

### setRouteVar

Call the `setRouteVar` method to set a variable within the scope of the current route. 

```typescript
// definition
type setRouteVar = <V>(varName: string, varValue: V | (msg: Msg, context?: IMessageContext) => V) => RouteClass
```

```typescript
// example.ts (continued)
route.setRouteVar('example', 'test')
```

_[Back to top](#developing-interface-channels-in-oop-style)_

### getRouteVar

Call the `getRouteVar` method to get a variable within the scope of the current route. 

```typescript
// definition
type getRouteVar = <V>(varName: string, cb: (val: V, msg: Msg, context: IMessageContext) => void | Msg | boolean) => RouteClass
```

```typescript
// example.ts (continued)
route.getRouteVar('example', (test) => console.log(test))
```

_[Back to top](#developing-interface-channels-in-oop-style)_

### send

Call the `send` method to get a variable within the scope of the current route. 

```typescript
// definition
type send = (method: 'tcp', host: string, port: number) => RouteClass
```

```typescript
// example.ts (continued)
route.send('tcp', 'localhost', 5505)
```

_**NOTE**: Currently only TCP clients are supported. In a future release, there will be additional methods to senders, writers, and callback messages._

_[Back to top](#developing-interface-channels-in-oop-style)_

## CompleteClass

The `CompleteClass` is returned by both the `route` and `routes` methods of the `IngestionClass`. This class currently has the following methods:

### run

Call the `run` method to start the channel on the server. This method does not support any arguments

```typescript
// example.ts (continued)
comp.run()
```

_[Back to top](#developing-interface-channels-in-oop-style)_

### export

Call the `export` method to export the `ChannelConfig` JSON script created. This method does not support any arguments

```typescript
// example.ts (continued)
comp.export()
```

_[Back to top](#developing-interface-channels-in-oop-style)_

### msg

(_FUTURE_) Call the `msg` method to define a callback function to call for at the end of each route.

_**NOTE**: This will be used in a future release with non listening channel configs, such as a message pass-through or one-time file read_

```typescript
// definition
type msg = (callback: (msg: Msg, context: IMessageContext) => void) => void
```

```typescript
// example.ts (continued)
let msg: string | undefined
comp.msg((m) => {
  msg = m.toString()
})
```

_[Back to top](#developing-interface-channels-in-oop-style)_
