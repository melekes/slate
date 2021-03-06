---
title: API Reference

language_tabs:
  - shell
  - go

toc_footers:
  - <a href='https://github.com/tripit/slate'>Documentation Powered by Slate</a>

includes:
  - errors

search: true
---

# Introduction

Tendermint supports the following RPC protocols:

* URI over HTTP
* JSONRPC over HTTP
* JSONRPC over websockets

Tendermint RPC is build using [our own RPC library](https://github.com/tendermint/tendermint/tree/master/rpc/lib). Documentation and tests for that library could be found at `tendermint/rpc/lib` directory.

## Configuration

Set the `laddr` config parameter under `[rpc]` table in the $TMHOME/config.toml file or the `--rpc.laddr` command-line flag to the desired protocol://host:port setting.  Default: `tcp://0.0.0.0:46657`.

## Arguments

Arguments which expect strings or byte arrays may be passed as quoted strings, like `"abc"` or as `0x`-prefixed strings, like `0x616263`.

## URI/HTTP

> Example request:

```bash
curl 'http://localhost:46657/broadcast_tx_sync?tx="abc"'
```

> Response:

```json
{
  "error": "",
  "result": {
    "hash": "2B8EC32BA2579B3B8606E42C06DE2F7AFA2556EF",
    "log": "",
    "data": "",
    "code": 0
  },
  "id": "",
  "jsonrpc": "2.0"
}
```

The first entry in the result-array (`96`) is the method this response correlates with. `96` refers to "ResultTypeBroadcastTx", see [responses.go](https://github.com/tendermint/tendermint/blob/master/rpc/core/types/responses.go) for a complete overview.

## JSONRPC/HTTP

JSONRPC requests can be POST'd to the root RPC endpoint via HTTP (e.g. `http://localhost:46657/`).

> Example request:

```json
{
  "method": "broadcast_tx_sync",
  "jsonrpc": "2.0",
  "params": [ "abc" ],
  "id": "dontcare"
}
```

## JSONRPC/websockets

JSONRPC requests can be made via websocket. The websocket endpoint is at `/websocket`, e.g. `http://localhost:46657/websocket`.  Asynchronous RPC functions like event `subscribe` and `unsubscribe` are only available via websockets.


## More Examples

See the various bash tests using curl in `test/`, and examples using the `Go` API in `rpc/client/`.

# Endpoints

## Get the list

An HTTP Get request to the root RPC endpoint (e.g. `http://localhost:46657`) shows a list of available endpoints.

```bash
curl 'http://localhost:46657'
```

> Response:

```plain
Available endpoints:
http://localhost:46657/abci_info
http://localhost:46657/dump_consensus_state
http://localhost:46657/genesis
http://localhost:46657/net_info
http://localhost:46657/num_unconfirmed_txs
http://localhost:46657/status
http://localhost:46657/unconfirmed_txs
http://localhost:46657/unsafe_flush_mempool
http://localhost:46657/unsafe_stop_cpu_profiler
http://localhost:46657/validators

Endpoints that require arguments:
http://localhost:46657/abci_query?path=_&data=_&prove=_
http://localhost:46657/block?height=_
http://localhost:46657/blockchain?minHeight=_&maxHeight=_
http://localhost:46657/broadcast_tx_async?tx=_
http://localhost:46657/broadcast_tx_commit?tx=_
http://localhost:46657/broadcast_tx_sync?tx=_
http://localhost:46657/commit?height=_
http://localhost:46657/dial_seeds?seeds=_
http://localhost:46657/subscribe?event=_
http://localhost:46657/tx?hash=_&prove=_
http://localhost:46657/unsafe_start_cpu_profiler?filename=_
http://localhost:46657/unsafe_write_heap_profile?filename=_
http://localhost:46657/unsubscribe?event=_
```

## Tx

```shell
curl "http://localhost:46657/tx?hash=0x2B8EC32BA2579B3B8606E42C06DE2F7AFA2556EF"
```

```go
client := client.NewHTTP("tcp://0.0.0.0:46657", "/websocket")
tx, err := client.Tx([]byte("2B8EC32BA2579B3B8606E42C06DE2F7AFA2556EF"), true)
```

> The above command returns JSON structured like this:

```json
{
  "error": "",
  "result": {
    "proof": {
      "Proof": {
        "aunts": []
      },
      "Data": "YWJjZA==",
      "RootHash": "2B8EC32BA2579B3B8606E42C06DE2F7AFA2556EF",
      "Total": 1,
      "Index": 0
    },
    "tx": "YWJjZA==",
    "tx_result": {
      "log": "",
      "data": "",
      "code": 0
    },
    "index": 0,
    "height": 52
  },
  "id": "",
  "jsonrpc": "2.0"
}
```

Returns a transaction matching the given transaction hash.

### Query Parameters

Parameter | Default | Required | Description
--------- | ------- | -------- | -----------
hash      | nil     | true     | The transaction hash
prove     | false   | false    | Include a proof of the transaction inclusion in the block in the result (optional, default: false)

### Returns

- `proof`: the `types.TxProof` object
- `tx`: `[]byte` - the transaction
- `tx_result`: the `abci.Result` object
- `index`: `int` - index of the transaction
- `height`: `int` - height of the block where this transaction was in
