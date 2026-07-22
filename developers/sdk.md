# SDK reference

`@openajo/sdk` is the typed TypeScript client for both contracts. It ships as TypeScript source; consumers transpile it.

```ts
import {
  getCircle, getMembers, getMember, getCycleDeadline, getTotalCircles, getReputation,
  createCircle, joinCircle, leaveCircle, cancelCircle, contribute, settleCycle,
  decodeEvent, formatUnits, parseUnits,
  type OpenAjoConfig, type Circle, type MemberState, type Reputation,
} from "@openajo/sdk";
```

## Configuration

Every call takes a config object:

```ts
const config: OpenAjoConfig = {
  rpcUrl: "https://soroban-testnet.stellar.org",
  networkPassphrase: "Test SDF Network ; September 2015",
  circleContractId: "CCLVOHGHDH32GWFAMCEMVHLNJSF6ENVHERYWU2OHUYWWLAOKLVR3HGKS",
  reputationContractId: "CDXPH2PYUTRW7GV57X6CJH3E3JOPROSC23NXPMAXOO3EOBI5UTCB2GTQ",
  readSource: "G...", // any funded account; source for read simulations
};
```

`readSource` is only used to build simulation transactions. No secret is involved and nothing is submitted.

## Reads

Reads run as RPC simulations — free, instant, no signature.

```ts
const circle = await getCircle(config, 0);
// { creator: "G...", token: "C...", contribution: 100000000n, deposit: 150000000n,
//   size: 3, periodSecs: 604800n, status: "Active", startedAt: 1784…n, currentCycle: 1 }

const members = await getMembers(config, 0);      // string[] in join order
const me      = await getMember(config, 0, addr); // { depositRemaining, received, defaulted }
const due     = await getCycleDeadline(config, 0);// bigint unix seconds
const total   = await getTotalCircles(config);    // number
const rep     = await getReputation(config, addr);// { completed, defaulted }
```

Failed simulations throw `ContractCallError` with the decoded contract error:

```ts
import { ContractCallError, CircleError } from "@openajo/sdk";

try {
  await getCircle(config, 999);
} catch (e) {
  if (e instanceof ContractCallError) {
    e.code;    // 3
    e.message; // "That circle doesn't exist."
    e.code === CircleError.NotFound; // true
  }
}
```

## Writes

Writes take a `SignFn` — the SDK never handles keys.

```ts
export type SignFn = (xdrBase64: string, networkPassphrase: string) => Promise<string>;
```

### Browser (Freighter)

```ts
import { signTransaction } from "@stellar/freighter-api";

const freighterSigner: SignFn = async (xdr, networkPassphrase) => {
  const res = await signTransaction(xdr, { networkPassphrase });
  if (res.error) throw new Error(String(res.error));
  return res.signedTxXdr;
};
```

### Server (keypair — fees only, e.g. a settle crank)

```ts
import { keypairSigner } from "@openajo/sdk";
const sign = keypairSigner(process.env.CRANK_SECRET!);
```

### Calls

```ts
await createCircle(config, {
  creator: addr,
  token: TOKEN_ID,
  contribution: parseUnits("10"),   // 10 tokens -> 100000000n
  deposit:      parseUnits("15"),
  size: 5,
  periodSecs: 604800n,              // weekly
}, sign);

await joinCircle(config, circleId, addr, sign);
await leaveCircle(config, circleId, addr, sign);
await cancelCircle(config, circleId, creator, sign);
await contribute(config, circleId, addr, sign);
await settleCycle(config, circleId, anyFundedAccount, sign);
```

Each returns `{ hash }` once the transaction is confirmed. Internally: build → `prepareTransaction` (simulate, attach auth and resource fees) → sign → submit → poll to inclusion.

## Amounts

Always `bigint` raw units. Convert only at the edges:

```ts
formatUnits(123456789n);   // "12.3456789"
formatUnits(100000000n);   // "10"
parseUnits("12.3456789");  // 123456789n
parseUnits("1.00000001");  // throws — more precision than 7 decimals
```

Never use `Number()` on a raw amount.

## Events

```ts
import { decodeEvent, type OpenAjoEvent } from "@openajo/sdk";

for (const ev of page.events) {
  const decoded: OpenAjoEvent | null = decodeEvent(ev.topic, ev.value);
  if (!decoded) continue;               // not an OpenAjo event
  if (decoded.kind === "payout") {
    decoded.recipient; decoded.amount; decoded.cycle;
  }
}
```

`OpenAjoEvent` is a discriminated union over `kind`: `create`, `join`, `start`, `contrib`, `slash`, `default`, `payout`, `complete`, `cancel`, `rep_complete`, `rep_default`. See [Events](../contracts/events.md) for the on-chain shapes.
