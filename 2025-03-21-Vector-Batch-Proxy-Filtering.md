# Disclosure 2025-03-14 VectorX Bridge Proxy/Multisig Transaction Filtering

## Disclosure

On March 13, the Avail team was notified of a potential vulnerability affecting the Bridge. The team responded to the report, which was done outside of our bug bounty program. We have confirmed the vulnerability and its impact the next day and created a fix in the Avail node's Vector pallet logic that we subsequently rolled out to preproduction and production environments on the same day.

Any non-batched bridge transaction that contained another transaction - was nested within a proxy  or multisig call, caused an issue. Leading to the transaction to be incorrectly filtered out. Meaning we won't be able to generate proofs for the call and it will be impossible to claim the funds when briding from AVAIL to Ethereum.

## Impact

The Vector pallet's`send_message`, when nested within a multisig or a proxy call, was not being included in the `bridgeRoot` , making it non-claimable on the Ethereum side of the bridge. When an operation that met this criteria was performed, it resulted in loss of funds and impacted the Avail → Ethereum side of the bridge. In total, this affected one transaction of a total of 100 AVAIL.

**This vulnerability did not affect normal bridge usage through [https://bridge.availproject.org/](https://bridge.availproject.org/). Funds transferred through the bridge UI were not at risk.** 

# Timeline

| Timestamp | Note |
| --- | --- |
| March 13, 2025 at 15:51:00 UTC | A potential issue with claiming was reported to Avail by a partner testing a specific bridge use case. |
| March 14, 2025 at 12:05:00 UTC | The team finished analyzing the details and impact of the issue, confirmed it as a vulnerability and started working on the fix in the Avail node's Vector pallet logic. |
| March 14, 2025 at 16:43:00 UTC | First version of the bug fixes were committed into a version of the fix. |
| March 14, 2025 at 17:25:00 UTC | Preliminary tests were done. Everything passed. |
| March 14, 2025 at 17:51:00 UTC | Spec version was increased in order to prepare for Hex network release. |
| March 14, 2025 at 17:56:00 UTC | Fix was deployed on an internal devnet version to perform testing. |
| March 14, 2025 at 19:42:00 UTC | Testing concluded and a green light was given to deploy the fix to theTuring testnet. |
| March 14, 2025 at 19:54:00 UTC | A technical committee proposal was created to deploy the fix to Turing network |
| March 14, 2025 at 20:35:00 UTC | The technical committee proposal was closed and Turing was updated to spec version 45 |
| March 14, 2025 at 20:35:00 UTC | Updating the Turing runtime resulted in a deployment of incompatible runtime to Turing, which made the Turing bridge not work on the way from Eth to Avail. This was agreed upon as part of a temporary measure to ship the fix for the current vulnerability. |
| March 14, 2025 at 22:20:00 UTC | Node [v2.2.5.4 was released](https://github.com/availproject/avail/releases/tag/v2.2.5.4) with the wasm that will be deploy on Mainnet. |
| March 14, 2025 at 22:52:00 UTC | A technical committee proposal was created to deploy the fix to Mainnet using the wasm from v2.2.5.4 release. |
| March 14, 2025 at 23:57:00 UTC | The proposal was closed was [closed](https://avail.subscan.io/tech/29?tab=votes) on block and took effect on block [109526](https://avail.subscan.io/block/1095296). Mainnet was updated to spec version 45. Fixing the vulnerability. |
| March 15, 2025 at 00:02:00 UTC | Hex has been updated to spec version 46. This enabled the bridge again in the Eth to Avail direction on Hex. |
| March 15, 2025 at 00:52:00 UTC | Node [v2.3.0.0-rc3 was released](https://github.com/availproject/avail/releases/tag/v2.3.0.0-rc3) with the wasm that will be deployed on Turing (spec version 46). |
| March 15, 2025 at 00:58:00 UTC | A technical committee proposal was created to deploy v2.3.0.0-rc3 to Turing. |
| March 15, 2025 at 01:03:00 UTC | A [transparency report](https://forum.availproject.org/t/transparency-report-16-critical-runtime-upgrade-filtering-vectorx-bridge-transactions/1665) was published on the Avail forum. |
| March 15, 2025 at 04:44:00 UTC | The Avail documentation was updated to better reflect the [transaction nesting limitations](https://docs.availproject.org/api-reference/avail-bridge-api). |
| March 15, 2025 at 12:50:00 UTC | Proposal was closed and Turing was updated to spec version 46. This enabled the bridge again in the Eth to Avail direction on Hex. |

## Causes

The `vector.send_message`logic in the Avail node was not handled correctly for scenarios when the call was made through a Proxy or Multisig call. This resulted in the the `send_message` call not being included in the `bridgeRoot` .

## Fixes

Filters in the node logic have been changed to correctly extract`data` from `call` if they are of the `vector.send_message` type. The logic has been changed to correctly support up to 2 levels of nesting only with proxy and multisig `vector.sendMessage` .

The Avail documentation was updated to better reflect the [transaction nesting limitations](https://docs.availproject.org/api-reference/avail-bridge-api).

## Conclusion

We would like to thank our partner who highlighted the issue with the logic that led to the vulnerability in the above-mentioned scenarios.
