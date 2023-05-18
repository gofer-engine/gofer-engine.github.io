### Decoding

HL7 messages can be decoded by simply passing the raw string message to the `Msg` constructor:

```typescript
const msg = new Msg(HL7_string)
```

To see the JSON decoded message, you can use the `json()` method:

```typescript
const json = new Msg(HL7_string).json()
console.log(JSON.stringify(json, undefined, 2))
```

You can pass an argument to the `json` method to normalize the JSON format

```typescript
const normalized = new Msg(HL7_string).json(true)
console.log(JSON.stringify(normalized, undefined, 2))
```

If you have a JSON object that you want to replace the decoded JSON object, you can use the `setMsg` method:

```typescript
const foo = new Msg(HL7_string)
const bar = new Msg('MSH|^~\\&|...')
bar.setMsg(foo.json())

console.log(bar.json())
```

### Encoding

To encode a message back to an HL7 string, you can use the 'toString' method on the `Msg` class:

```typescript
const msg = new Msg(HL7_string)
console.log(msg.toString())
```
