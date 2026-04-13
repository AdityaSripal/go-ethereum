# Approach B: Keep Gnosis Blob Defaults — Abandoned

## What this approach tried

Keep `BlobTxMinBlobGasprice=1e9`, `BlobTxMaxBlobs=2`, and `Default{Cancun,Prague,Osaka}BlobConfig` at Gnosis values (target=1, max=2). Fix all tests to expect these values.

## Why it was abandoned

`BlobTxMinBlobGasprice` is a package-level global in `params/protocol_params.go` used as the minimum blob gas price throughout the codebase. Changing it from `1` to `1e9` breaks tests pervasively:

- **blobpool tests** (~60 `makeUnsignedTx` calls with blobFeeCap=1): all rejected as below minimum. Fixing requires rewriting every test transaction and rebalancing seed account balances.
- **eth/gasprice**: `TestFeeHistory` panics.
- **eth/protocols/eth**: `TestGetPooledTransaction/blobTx` fails on min price.
- **eth/tracers**: `TestCallTracerNative/blobTx` and `TestPrestateTracer/blobTx` fail because `blobGasFeeCap < blobBaseFee`.

The eip4844 tests were fixable (explicit configs + updated expected values), and the forkid hack was guardable with a chain ID check. But the blobpool and transaction-level tests are designed around `minBlobGasPrice=1` so deeply that adapting them would be a massive, fragile rewrite.

## Approach A is better

Approach A reverts all blob globals to upstream Ethereum values. Gnosis/Chiado already load their correct blob params from chain spec JSON (`params/chainspecs/{gnosis,chiado}.json`) via `BlobScheduleConfig`. The globals only affect test configs and Ethereum mainnet, so reverting them is correct.
