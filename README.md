# gbx-protos

gRPC protos and the `gbx-protos` Rust crate for
[GBX](https://docs.gravityblocks.xyz/), an open marketplace for Solana
blockspace. Astralane's regional Gravity GBX broadcasters accept bundles and
fan them out to configured GBX builders, including Bifrost and Titan.

The API is wire-compatible with the subset of the
[Jito searcher API](https://github.com/jito-labs/mev-protos) used by GBX, so
existing Jito clients can use the same authentication and request flow after
changing the URL. Unsupported methods are omitted.

## Quick start

1. Register the public key that will authenticate to the Gravity GBX
   broadcaster.
2. Use `AuthService` to sign a challenge and obtain a bearer token.
3. Call `GetConnectedLeaders` to find upcoming slots whose leader is connected
   to GBX.
4. Pick the endpoint closest to your servers and send the signed transactions
   with `SendBundle`.

## Endpoints

| Region    | Endpoint                                      |
| --------- | --------------------------------------------- |
| Frankfurt | `http://fra-gbx.gateway.astralane.io:5064`    |
| Amsterdam | `http://ams-gbx.gateway.astralane.io:5064`    |
| London    | `http://lon-gbx.gateway.astralane.io:5064`    |
| Tokyo     | `http://jp-gbx.gateway.astralane.io:5064`     |

Every regional broadcaster fans accepted bundles out to its configured GBX
builders.

## Methods

| Method                        | Description                                              |
| ----------------------------- | -------------------------------------------------------- |
| `SendBundle`                  | Send a bundle of 1 to 5 signed transactions.             |
| `SendTransaction`             | Send a single signed transaction.                        |
| `GetConnectedLeaders`         | Connected validators and their upcoming leader slots.    |
| `GetConnectedLeadersRegioned` | The same, grouped by region.                             |
| `GetRegions`                  | Your current region and all available regions.           |

There is no bundle result stream; check landing on chain.

## Bundles

A bundle lands all or nothing, in order. Optional fields on top of Jito's
`Bundle`:

- `dropping_indices`: transactions that may be dropped if they fail.
- `slot`: only land in this slot.
- `target_accounts`, `target_bundle_id`: see `protos/bundle.proto`.

## Tipping

The Gravity GBX broadcaster does not impose a minimum tip or select a builder's
tip account. Include the fee or tip required by the downstream builders you
want to target.

## Rate limits

Limits are configured per authenticated public key. For access or a higher
limit, reach out on [Discord](https://discord.gg/hnerZ7dhTk).

## Using the crate

```toml
[dependencies]
gbx-protos = { git = "https://github.com/Astralane/gbx-protos" }
tonic = { version = "0.14", features = ["tls-ring", "tls-webpki-roots"] }
```

[`examples/send_bundle.rs`](examples/send_bundle.rs) prints the next connected
leaders, then sends the given signed transactions (raw or base64 files) as one
bundle:

```sh
cargo run --example send_bundle -- http://ams-gbx.gateway.astralane.io:5064
```

Reuse one connection rather than reconnecting per bundle.

Other languages: compile `protos/` with any protobuf toolchain, or use a Jito
client. With `grpcurl`:

```sh
grpcurl -plaintext -import-path protos -proto searcher.proto \
  ams-gbx.gateway.astralane.io:5064 searcher.SearcherService/GetConnectedLeaders
```

## Development

Generated bindings are checked in under `src/generated`. After editing
`protos/*.proto`, regenerate them with:

```sh
REGENERATE_PROTO=1 cargo build
```
