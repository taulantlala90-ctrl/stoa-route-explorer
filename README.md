# Stoa route explorer

A dependency-free browser dashboard reading the Stoa Router's `Routed` events on Robinhood Chain (ID 4663). It reports all routed swaps, distinct sender addresses, counts per venue, recent transactions, USDG-sided volume, and an estimated total USD volume using current prices. USDG-sided volume is not total dollar volume: swaps without USDG are excluded. The dashboard never sends swaps, connects a wallet, or publishes a wallet address.

## Run

Serve this directory as static files (for example, `python -m http.server` from this directory) and open the local HTTP URL. The remote RPC must be reachable from the visitor's browser. The application currently requests all events in one JSON-RPC request, so RPC outages, rate limits, response caps and historical range limits may prevent it loading.

## Source and query

Router: `0x2F2804fad735a38015650f32f95Ad5E08D6B3eE5`.
Contract event definition: https://github.com/StoaDex/Stoa/blob/main/contracts/src/StoaRouter.sol
Deployment block and known venue addresses: https://github.com/StoaDex/Stoa/blob/main/web/lib/deployments-4663.json
RPC: https://rpc.mainnet.chain.robinhood.com
Method: `eth_getLogs`, `fromBlock: 114244443`, `toBlock: latest`, router address as above, event topic `0x2bbce25631e8fa58085b7ea5929726c3acdf2df8d6b1f099eb32fb7425a4f6da`.
The indexed first topic after event signature is sender. Seven 32-byte words in event data are tokenIn, tokenOut, amountIn, amountOut, mid, venue1 and venue2. The USDG contract has 6 decimals.

## Check

On 2026-11-20 the live RPC returned 32 events, 19 distinct senders, and six identified/unknown venue entries. The live RPC test can be run with `node test-live.cjs`. It uses a minimal mocked DOM and live network requests; a rendered browser view was separately checked.

## Limitations

Volume for non-USDG pairs is an estimate based on current prices, not historical execution values. If a price source is unavailable, the page reports unpriced swaps. The public RPC may limit historical requests as the number of swaps grows.

## Valuation details

A swap is valued once using its USDG input or output when present. Otherwise, the ETH or WETH side is valued at CoinGecko’s current ETH/USD spot price. Other pairs use the first usable token side with the current USD price from the Robinhood Chain pair with greatest available liquidity on DexScreener; decimals are read from the token’s on-chain `decimals()` call. Current prices are not historical execution prices. The dashboard counts swaps for which no price or decimals could be obtained, excludes them from the estimate, and reports the missing count. Third-party availability, API rate limits, low liquidity, price slippage, token anomalies, and JavaScript floating point can affect results. The USDG-only total is shown separately. On a live check November 20, 2026, 32 swaps were priced and the estimated total was $628.83; values change with the market and are not expected to match Stoa’s numbers exactly.
