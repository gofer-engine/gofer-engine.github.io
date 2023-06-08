[🏠 Gofer Engine](https://gofer-engine.github.io/) > [Msg Class](./index.md) > Sub Classes

## Sub Classes

**NOTE: THIS PAGE IS STILL A WORK IN PROGRESS**

### Seg, Segs & Seg[]

- `Seg` — returned when there is only a single Segment by `name`
- `Segs` — returned when there are multiple Segments by `name` if no `iteration` is defined

```ts
Msg.getSegment(name: string, iteration?: number) => Seg | Segs
```

- `Seg[]` returned `Seg` classes in an array. _**NOTICE** These will be 0-based index._

```ts
Msg.getSegments(name: string) => Seg[]
```

### Fld, Flds, & Fld[]

- Fld (`Seg.getField(index: number, iteration?: number)`)

- Flds

- Fld[] (`Seg.getFields(index: number)`) returned `Fld` classes in an array. _**NOTICE** These will be 0-based index._

### Rep[]

- Rep[] (`Fld.getRepetitions()`) returned `Rep` classes in an array. _**NOTICE** These will be 0-based index._

### Cmp, Cmps, & Cmp[]

- Cmp returned when there is a single component

```ts
Fld.getComponent(index?: number)
Flds.getComponent(index?: number)
Rep.getComponent(index?: number)
```

- Cmps (`Field.getComponent(index?: number)`) returned when there are multiple components
- Cmp[] (`Field.getComponents(index: number)`) returned `Cmp` classes in an array. _**NOTICE** These will be 0-based index._

### Sub, Subs, & MultiSubs

- SubComponent (`Component.getSubComponent(index: number)`)

Each of the above subclasses exposes the following methods:

- `json(strict?: boolean)` — 
- `toString()` — 

The `Seg` class exposes an additional `getName` method that returns the segment name string.