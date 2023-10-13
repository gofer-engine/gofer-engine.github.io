[🏠 Gofer Engine](./index.md) > Creating an HL7 Messenger

# Creating an HL7 Messenger

Are you building a Node.JS such as a Next.JS application or GraphQL.js API and needing to integrate directly with HL7 interfaces? We have included messengers to make creating and sending HL7 simple once and for all.

Here is a detailed example of using Gofer Engine to send `ADT_A20` messages within a Next.JS API page.

```ts
import type { NextApiRequest, NextApiResponse } from 'next'
import gofer from `@gofer-engine/engine`;

const table0116_cust = {
  Cleaning: 1,
  Clean: 2
} as const

interface BedStatusUpdateRequest extends NextApiRequest {
  body: {
    operator: string;
    bed: string;
    status: keyof typeof table0116_cust;
  };
}

const table0008 = {
  AA: "Accepted",
  AE: "Errored",
  AR: "Rejected",
  CA: "Commit Accepted",
  CE: "Commit Errored",
  CR: "Commit Rejected"
} as const

type ResponseData = {
  ack: "Unknown" | typeof table0008[keyof typeof table0008];
}

const EHR_HL7_IP = '192.168.1.200';
const EHR_HL7_PORT = 5600

const [messenger] = gofer.messenger((route) => {
  // TODO: Configure message logging to outbound MSG and inbound ACK
  return route.send('tcp', EHR_HL7_IP, EHR_HL7_PORT)
})

/**
 * @param operator - Operator ID needs to be set in Meditech in EVS Staff Dictionary. Path: Administrative > Registration > Dictionary (REG Registration) > EVS Staff
 * @param bed - Bed Location (aka Telephony Ext) needs to be set in Meditech in MIS Room Dictionary. Path: Info Systems > MIS > Dictionaries > Administrative > Room
 * @param status - Bed Status. Table 0116 from Meditech: 1=Cleaning in Process, 2=Clean. Any other value will be rejected.
 */ 
const sendBedUpdate = async (
  operator: string,
  bed: 
  status: 1 | 2,
) => {
  const now = new Date().toISOString().replace(/-|:|T/g, '').slice(0, 12);
  const ack = await messenger((msg) => {
    return msg
      .set('MSH-3', 'YOUR APP NAME')
      .set('MSH-4', 'YOUR FACILITY')
      .set('MSH-5', 'RECEIVING APP NAME')
      .set('MSH-6', 'RECEIVING FACILITY')
      .set('MSH-7', now)
      .set('MSH-9.1', 'ADT')
      .set('MSH-9.2', 'A20')
      .set('MSH-9.3', 'ADT_A20')
      .set('MSH-10', id)
      .set('MSH-11', 'T')
      .set('MSH-12', '2.4')
      .addSegment('EVN')
      .set('EVN-2', now)
      .set('EVN-5', operator)
      .addSegment(['NPU', bed, status])
  })
}

export default function handler(
  req: BedStatusUpdateRequest,
  res: NextApiResponse<ResponseData>
) {
  if (req.method !== 'POST') {
    res.status(405).text('Unsupported Method')
  } else {
    const { operator, bed, status } = req.body
    const mappedStatus = table0116_cust?.[status]
    // TODO: add input validation
    const response = await sendBedUpdate(operator, bed, mappedStatus)
    res.status(200).json({
      ack: table0008?.[response.get('MSA.1')] || "Unknown"
    })
  }
}
```