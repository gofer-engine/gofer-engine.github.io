[🏠 Gofer Engine](https://gofer-engine.github.io/) > [Msg Class](./index.md) > Extrapolating

### Extrapolating

HL7 uses paths to reference data inside of the HL7 message structure. The path is a string that is a combination of the segment name, segment iteration index, field index, field iteration index, component index, and subcomponent index. Iteration indexes are surrounded by brackets (`[...]`). The other path parts are separated by either a period (`.`) or a dash (`-`). The path is 1-indexed, meaning the first segment iteration is `1`, the first field is `1`, the first field iteration is `1`, the first component is `1`, and the first subcomponent is `1`. The segment name is 3 upper case characters. A path can be specific down to the subcomponent level with optional iteration indexes or can be as general as just the segment name. The following are all valid paths (_may not be valid HL7 schemed messages)):

`MSH`, `MSH-3`, `MSH.7`, `MSH.9.1`, `MSH.9-2`, `MSH.10`, `STF-2.1`, `STF-2[2].1`, `STF-3.1`, `STF-11[2]`, `LAN[1]`, `LAN[2].3`, `LAN[3].6[1].1`

The `Msg` class provides methods to structure and destructure paths.

```ts
type Paths = {
  segmentName?: string
  segmentIteration?: number
  fieldPosition?: number
  fieldIteration?: number
  componentPosition?: number
  subComponentPosition?: number
}
Msg.paths = (path?: string) => Paths
Msg.toPath = (path?: Paths) => string
```

Iteration indexes can be defined as `[1]` even for non-iterative paths. Simplified 1 based paths are also supported even if the value is not deeply nested. For example `ZZZ[1]-1[1].1` might reference the same value (`Source`) as the following: `ZZZ[1]-1[1]`, `ZZZ[1]-1`, `ZZZ-1`, `ZZZ-1[1]`, and `ZZZ-1.1` in the the following HL7 messages:

```hl7
MSH|^~\&|HL7REG|UH|HL7LAB|CH|200702280700||PMU^B01^PMU_B01|MSGID002|P|2.5.1|
EVN|B01|200702280700|
STF||U2246^^^PLW~111223333^^^USSSA^SS|HIPPOCRATES^HAROLD^H^JR^DR^M.D.|P|M|19511004|A|^ICU|^MED|(555)555-1003X345^C^O~(555)555-3334^C^H~(555)555-1345X789^C^B|1003 HEALTHCARE DRIVE^SUITE 200^ANNARBOR^MI^98199^H~3029 HEALTHCARE DRIVE^^ANNARBOR^MI^98198^O|19890125^DOCTORSAREUS MEDICAL SCHOOL&L01||PMF88123453334|74160.2326@COMPUSERV.COM|B
PRA||^HIPPOCRATES FAMILY PRACTICE|ST|I|OB/GYN^STATE BOARD OF OBSTETRICS AND GYNECOLOGY^C^19790123|1234887609^UPIN~1234987^CTY^MECOSTA~223987654^TAX~1234987757^DEA~12394433879^MDD^CA|ADMIT&T&ADT^MED&&L2^19941231~DISCH&&ADT^MED&&L2^19941231|
AFF|1|AMERICAN MEDICAL ASSOCIATION|123 MAIN STREET^^OUR TOWN^CA^98765^U.S.A.^M |19900101|
LAN|1|ESL^SPANISH^ISO639|1^READ^HL70403|1^EXCELLENT^HL70404|
LAN|2|ESL^SPANISH^ISO639|2^WRITE^HL70403|2^GOOD^HL70404|
LAN|3|FRE^FRENCH^ISO639|3^SPEAK^HL70403|3^FAIR^HL70404|
EDU|1|BA^BACHELOR OF ARTS^HL70360|19810901^19850601|YALE UNIVERSITY^L|U^HL70402|456 CONNECTICUT AVENUE^^NEW HAVEN^CO^87654^U.S.A.^M|
EDU|2|MD^DOCTOR OF MEDICINE^HL70360|19850901^19890601|HARVARD MEDICAL SCHOOL^L |M^HL70402|123 MASSACHUSETTS AVENUE^CAMBRIDGE^MA^76543^U.S.A.^M|
ZZZ|Source|HL7 Version 2.5.1 Standard^Chapter&15&Personnel Management^Section&5&Example Transactions^Page&15-40^Date&200704
```

The `Msg` class exposes the `get` method that accepts a string which can be comprised of the following:

- Segment Name
- Segment Repition Index
- Field Index
- Field Repition Index
- Component Index
- Sub-Component Index

Repetition Indexes are optional and can be omitted. If omitted on a repeating segment/field, then an array of values will be returned.

For example to get all LAN segments you can use the get path `LAN`.

If you wanted just one of the segments, you would add in the repetition index, for example, `LAN[`2]` would return the second LAN segment.

Repetition Indexes are always syntactically wrapped in square brackets. e.g. `[2]`

Field, Component, and Sub-Component are optional and can be omitted. If omitted and the field/component is made up of smaller units, then an array of values will be returned.

For example, to get all the components of the language codes in the 1st LAN segment you can use the get path `LAN[1]-2`

Field, Component, and Sub-Component are always prefixed with either a dot or a hyphen. e.g. `.2` or `-2`

If there is only a single larger component, then the smaller divisions can be omitted, but if you specify the 1st smaller division, then it will return the larger component itself.

For example, `EVN-1.1.1` will return the same as `EVN-1.1` or `EVN-1`

Fields can also be repetitive and can be accessed by using the field repetition index. For example, `STF-10[1]` will return the 1st repetition of the 10th field in the SFT segment.

Compare how these different extracted paths compare:

```ts
import Msg from './src/'
import fs from 'fs'

const HL7 = fs.readFileSync('./sample.hl7', 'utf8')

const msg = new Msg(HL7)

console.log('STF-10[1].1:', msg.get('STF-10[1].1'))   // (555)555-1003X345
console.log('STF[1]-10[1]:', msg.get('STF[1]-10[1]')) // ['(555)555-1003X345','C','O']
console.log('STF-10[1]:', msg.get('STF-10[1]'))       // ['(555)555-1003X345','C','O']
console.log('STF-10.1:', msg.get('STF-10.1'))         // ['(555)555-1003X345','(555)555-3334','(555)555-1345X789']
console.log('STF.10.1:', msg.get('STF.10.1'))         // ['(555)555-1003X345','(555)555-3334','(555)555-1345X789']
console.log('LAN:', msg.get('LAN'))                   // [Seg{ ... },Seg{ ... },Seg{ ... }]
console.log('LAN[2]:', msg.get('LAN[2]'))             // Seg{ ... }
console.log('LAN-2.1:', msg.get('LAN-2.1'))           // ['ESL','ESL','FRE']
console.log('LAN[1]-2.1:', msg.get('LAN[1]-2.1'))     // 'ESL'
console.log('LAN-2:', msg.get('LAN-2'))               // [['ESL','SPANISH','ISO639'],['ESL','SPANISH','ISO639'],['FRE','FRENCH','ISO639']]
console.log('ZZZ-2.2', msg.get('ZZZ-2.2'))            // ['Chapter','15','Personnel Management' ]
console.log('ZZZ-2.2.1', msg.get('ZZZ-2.2.1'))        // 'Chapter'
```

Unless a path is fully defined including all repetition indexes, then the type returned could be an array or a string.

If the path is only a segment name, then either a single Segment (`Seg`) class or an array of Segment (`Seg`) classes will be returned. See [SegmentClass](./sub-classes.md#segmentclass). If you want to be sure to get back a singular string value, then you should use the most specific path possible. For example, if you want to get the first component of the first field of the first segment iteration of the `MSH` segment, then you should use the path `MSH[1].1.1`. If you use the path `MSH.1.1`,  and there are multiple MSH segments, then you will get back an array of strings, one for each segment iteration.

You can use wrap the `get` method around the `toPath` method, to specify path components individually instead of a concatenated string.

```typescript
const event = msg.get(msg.toPath({
  segmentName: 'MSH',
  segmentIteration: 1, // could be left out
  fieldPosition: 9,
  fieldIteration: 1, // could be left out
  componentPosition: 2,
  subComponentPosition: 1, // could be left out
}))
```
