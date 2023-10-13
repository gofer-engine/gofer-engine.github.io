[🏠 Gofer Engine](https://gofer-engine.github.io/) > [Channel Workflows](./index.md) > Acknowledgement

# Acknowledgement

Acknowledgements are responses returned to the sender of a message. If a sender is using queuing, then an acknowledgement response promptly will inform the sender that the message was received whether accepted or rejected. This will let the sender move on to the next message in the queue.

The `ack` flow is an optional ingestion flow. If the server should respond to the source, then there should be an ack flow somewhere in the list of ingestion flows. If you do not add an ack config in your ingestion flow, and you are not utilizing source queuing, then no response will be returned to the sender.

The type `AckConfig` is:

```typescript
type AckConfig = {
  // Optional. Value to use in ACK MSH.3 field. Defaults to 'gofer Engine'
  application?: string
  // Optional. Value to use in ACK MSH.4 field. Defaults to ''
  organization?: string
  // Optional. Value to use in MSA-1 field. Defaults to 'AA'
  responseCode?: 'AA' | 'AE' | 'AR'
  // Optional. Text to use in ACK MSA.3. Defaults to ''
  text?: string
  // Optional. A function that accepts the ack MSG class, msg MSG class, and conext state object
  // and returns the ACK MSG class back. This allows for custom transformation of the ACK message.
  msg?: (ack: Msg, msg: Msg, context: IAckContext) => Msg
}
```

More information on [Msg](../msg-class/index.md)
More information on [Context Object](./context-object.md)

This configuration allows you to leave the defaults and return an ACK with the default application, organization, and text.

```typescript
// example.ts
import gofer from '@gofer-engine/engine'

gofer
  .listen('tcp', 'localhost', 5501)
  .name('Example CHannels with Configs')
  .ack()
  .run()

gofer
  .config([{
    name: 'Example Channels with Configs',
    source: {
      kind: 'tcp',
      tcp: {
        host: 'localhost',
        port: 9002,
      },
    },
    ingestion: [
      {
        kind: 'ack',
        ack: {},
      },
    ],
  }])
```

For a little more complex example, we can use data in the received message, and the filtered status to send back an ACK or NACK.

```typescript
// example.ts
import gofer, { AckConfig } from '@gofer-engine/engine'

const ackProcess: AckConfig['msg'] = (ack, msg, { filtered }) => {
  if (filtered) {
    return ack
      .set('MSA-2', 'AR')
      .set('MSA-3', `${msg.get('PID-3[1].1')} was filtered`)
  }
  return ack.set('MSA-3', `${msg.get('PID-3[1].1')} was accepted`)
}

gofer
  .listen('tcp', 'localhost', 5501)
  .name('Example CHannels with Configs')
  .ack({
    msg: ackProcess
  })
  .run()

gofer
  .config([{
    name: 'Example Channels with Configs',
    source: {
      kind: 'tcp',
      tcp: {
        host: 'localhost',
        port: 9002,
      },
    },
    ingestion: [
      {
        kind: 'ack',
        ack: {
          msg: ackProcess
        },
      },
    ],
  }])
```
