[🏠 Gofer Engine](https://gofer-engine.github.io/) > [Msg Class](./index.md) > Transforming

## Transforming

You can use the following methods to transform the message. Each method returns the self-class instance, so you can chain the methods together as needed.

### addSegment

`addSegment(segment: string | Segment | Seg | Segs, after?: number | string)` — Adds a HL7 encoded segment string or Segment json or Seg(s) class(es) after the indicated position in the message.

```ts
// parses the HL7 segment(s) and adds the segment(s) at the end of the message
msg.addSegment('ZZZ|Test')

// add the simplified json segment to the beginning of the message
const segment = ['MSH', '|', '^~\\&',,,,,,,['PMU','B01','PMU_B01'],,,'2.5.1',]
msg.addSegment(segment, 0)

// add the segment class after the first OBX segment
const seg = new Seg(['NTE', 1, null, 'Some Note'])
msg.addSegment(seg, 'OBX')

// parse and adds multiple segments after the second OBX segment
const notes = `NTE|1||Multi-line
NTE|1||Note`
msg.addSegment(notes, 'OBX[2]')

// add multiple simplified json segments after the SPM, OBR, OBX[2], TCD segment sequence
const noteArray = [['NTE', 1, , 'Foo'], ['NTE', 1, , 'Bar']]
msg.addSegment(noteArray, 'SPM:OBR:OBX[2]:TCD')
```

Note: If the path sequence is not found then an error will be returned.

### transform

`transform(limit: { restrict: IMsgFieldList, remove: IMsgFieldList })` — Transforms the message by restricting to only certain elements and/or removing certain elements. See the comments in the example below.

```ts
msg.transform({
  restrict: {
    MSH: () => true, // if function equates to true, keep whole segment
    LAN: 3, // an integer keeps only the nth iteration of the segment (LAN[3])
    ZZZ: { // an object keeps only certain fields of the segment
      1: true, // true keeps the entire field as is (ZZZ-1)
      2: [1, 5] // an array keeps only certain components of the field (ZZZ-2.1, ZZZ-2.5)
    },
    STF: {
      2: [1, 4],
      3: [], // an empty array keeps no components, same as if key was undefined
      4: [2],
      5: true,
      10: 1, // an integer keeps only the nth repeition of the field
      11: (f) => f?.[5] === 'O' // NOICE! `f` here is 0-indexed. This is keeping only the repetition that has STF-11.6 === 'O'
    },
    EDU: true, // if true, keep whole segment as is
    // Any segments not specifically listed in a `restrict` object are removed.
  },
  remove: {
    LAN: 2,
    EDU: {
      1: true, // deletes EDU-1
      2: [3], // deletes EDU-2.3
      3: (f) => {
        if (Array.isArray(f) && typeof f[0] === 'string')
          return f[0] > '19820000' // NOTICE! `f` here is 0-indexed. This looks at EDU-3.1
        return false
      }, // A function deletes when returns true. This is deleting EDU-3 when EDU-3.1 is greater than the year 1982
      4: 2 // deletes EDU-4[2]
    }
  }
})
```

### copy

`copy(fromPath: string, toPath: string)` — Copies the value from one path to another. Copy allows for deep copy copying repeating fields, components, and sub-components that may exist.

```ts
msg.copy('ZZZ-2', 'MSH-3')
```

### move

`move(fromPath: string, toPath: string)` — Moves the value from one path to another. Move is the same as copying the value to a temporary variable, deleting the value, writing the variable to the new path, and then clearing the temporary variable.

```ts
msg.move('LAN[1]-3[2]', 'LAN[1]-4')
```

### delete

`delete(path: string)` — Removes the value at the given path

```ts
msg.delete('EDU-6.6').delete('ZZZ')

```

### set

`set(path: string, value: string)` — Sets the value at the given path

```ts
msg.set('LAN[3]-4.1', '2')
msg.set('LAN[3]-4.2', 'GOOD')
```

### setJSON

`setJSON(path: string, value: MsgValue)` — Sets the value at the given path with a JSON object in case of sub-items.

```ts
msg.setJSON('LAN-4', ['3', 'FAIR', 'HL70404'])
```

### map

`map(path: string, dictionary: string | Record<string, string> | string[] | <T>(val: T, index: number) => T, { iteration: boolean })` — Sets the value at the given path with a mapped value. If the map is a string, then the value will be replaced with the map. If the map is a key-value object, then the value will be replaced with the value of the key-value object where the existing value matches the key. If the map is an array, then the value will be replaced with the value of the array at the index of the value converted to an integer (1-based indexing). If the map is a function, then the value will be replaced with the return value of the function. The function will be passed the value and the index of the value (1-based indexing). If the iteration option is set to true, then the function will be called for each iteration of the path. If the iteration option is set to false, then the function will only be called once for the path. Defaults to false.

```ts
// Dictionary Map an HL7 path using a key-value object.
msg.map('LAN-2.2', { ENGLISH: 'English' })

// Integer Map an HL7 path using an array. Array is treated as 1-based indexed.
msg.map('LAN-4.1', ['Excellent', 'Fair', 'Poor'])

// Function Map an HL7 path using a custom defined function that receives the original value and returns the new value.
msg.map('ZZZ-1', (orig) => orig.toUpperCase())

msg.map('LAN-4.2', {
  EXCELLENT: 'BEST',
  GOOD: 'BETTER',
  FAIR: 'GOOD',
})
```

### setIteration

`setIteration<Y>(path: string, map: Y[] | ((val: Y, i: number) => Y), { allowLoop: boolean })` — Sets the iteration of the path to the given map. If the map is an array, then the iteration will be set to the iterated index of the array (1-based). If the map is a function, then the iteration will be set to the return value of the function. The function will be passed the value and the index of the iteration (1-based indexing). If the allowLoop option is set to true and the array length is less than the iterations of the path, then the array will be looped over to fill the iterations. If the allowLoop option is set to false and the array length is less than the iterations of the path, then the remaining iterations will be set to empty. Defaults to false.

```ts
// Function Map an HL7 path using a custom defined function that receives the original value and returns the new value
msg.map('ZZZ-1', (orig) => orig.toUpperCase())

// Renumber LAN segment ids (LAN-1)
msg.setIteration<string>('LAN-1', (_v, i) => i.toString())

// Iteration Map an iterated HL7 path using an array. Useful for renumbering segments by type.
msg.setIteration('LAN-1', ['A', 'B', 'C'])
```

### Concatenating Transformers

The Transformer functions return the class itself allowing for the transformers to be chained in sequence.

```ts
msg
  .transform({
    restrict: {
      MSH: () => true, // if function equates to true, keep whole segment
      LAN: 3, // an integer keeps only the nth iteration of the segment (LAN[3])
      ZZZ: { // an object keeps only certain fields of the segment
        1: true, // true keeps the entire field as is (ZZZ-1)
        2: [1, 5] // an array keeps only certain components of the field (ZZZ-2.1, ZZZ-2.5)
      },
      STF: {
        2: [1, 4],
        3: [], // an empty array keeps no components, same as if key was undefined
        4: [2],
        5: true,
        10: 1, // an integer keeps only the nth repeition of the field
        11: (f) => f?.[5] === 'O' // f is 0-indexed. This is keeping only the repetition that has STF-11.6 === 'O'
      },
      EDU: true, // if true, keep whole segment as is
      // Any segments not specifically listed in a `restrict` object are removed.
    },
    remove: {
      LAN: 2,
      EDU: {
        1: true, // deletes EDU-1
        2: [3], // deletes EDU-2.3
        3: (f) => {
          if (Array.isArray(f) && typeof f[0] === 'string')
            return f[0] > '19820000'
          return false
        }, // A function deletes when returns true. This is deleting EDU-3 when EDU-3.1 is greater than the year 1982
        4: 2 // deletes EDU-4[2]
      }
    }
  })
  .addSegment('ZZZ|Engine|gofer')
  .copy('ZZZ-2', 'MSH-3')
  .set('ZZZ-1', 'Developer')
  .set('ZZZ-2', 'amaster507')
  .delete('LAN')
  .addSegment('LAN|1|ESL^SPANISH^ISO639|1^READ^HL70403~1^EXCELLENT^HL70404|')
  .addSegment('LAN|1|ESL^SPANISH^ISO639|2^WRITE^HL70403~2^GOOD^HL70404|')
  .addSegment('LAN|1|FRE^FRENCH^ISO639|3^SPEAK^HL70403~3^FAIR^HL70404|')
  .setIteration<string>('LAN-1', (_v, i) => i.toString())
  .move('LAN[1]-3[2]', 'LAN[1]-4')
  .move('LAN[2]-3[2]', 'LAN[2]-4')
  .move('LAN[3]-3[2]', 'LAN[3]-4')
  .map('LAN-4.2', {
    EXCELLENT: 'BEST',
    GOOD: 'BETTER',
    FAIR: 'GOOD',
  })
```
