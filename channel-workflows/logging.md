[🏠 Gofer Engine](https://gofer-engine.github.io/) > [Channel Workflows](./index.md) > Logging

# Logging

The logger is currently set up to log with `console.log`. In a future release, you will be able to hook into Gofer Engine and define your custom logger.

Logger is part of the [`Context`](./context-object.md). This allows you to use the logger from within any flow that includes the `context` such as a `filter` or `transformer`.

There are 4 log levels:

- `debug` should be used for development, testing, and debugging purposes only
- `info` should be used for your information only
- `warn` should be used for warnings that should be fixed, but are not show stoppers
- `error` should be used for show-stopping errors

The logger is fairly simple to use as seen in this example of a filter step:

```ts
// example.som
gofer
  .listen('tcp', 'localhost', 5501)
  .filter((msg, context) => {
    const isFemale = msg.get('PID-8')==='F'
    const { messageId, logger } = context
    if (!isFemale) {
      context.logger(`Msg ${messageId} was not a Female`, 'info')
    }
    return isFemale
  })
  .run()
```
