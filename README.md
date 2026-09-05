# VencordDashBeamWasm

Release distribution for the wasm bridge used by the **DashBeam** plugin in
[Loukious' Vencord fork](https://github.com/Loukious/Vencord).

The bridge is [dashbeam](https://github.com/tonyantony300/dashbeam)'s
`wasm-bridge` crate — iroh P2P file transfer compiled to WebAssembly
(Rust target `wasm32-unknown-unknown`, the standard browser-wasm target) —
plus a local patch:

- `receive_file` runs as a cancellable task (`n0_future::task::spawn`) with its
  `AbortHandle` stored in a static slot
- a new `cancel_receive()` export aborts the in-flight download

## Source

This repo vendors the full, self-contained source needed to rebuild the binary
(no reference to the dashbeam checkout):

```
wasm-bridge/          # the patched wasm-bindgen crate (our patch lives here)
  src/lib.rs
  .cargo/config.toml  # wasm32 linker + CC config
  Cargo.lock
engine/
  wasm-io/            # browser-friendly engine layer (sendme-wasm-io)
  protocol/           # transfer protocol (sendme-protocol)
```

Vendored from dashbeam commit `ccc1eb8` (upstream of merge PR #313), then
patched. Diffs vs upstream: `wasm-bridge/src/lib.rs` (cancel patch, `anyhow`
+ `n0-future` deps in `Cargo.toml`).

## Releases

Each release tags a build; the asset `wasm_bridge_bg.wasm` is what the plugin's
main process auto-downloads (versioned cache in `DATA_DIR/plugins/DashBeam`).
The JS glue (`wasm_bridge.js` / `.d.ts`) is vendored directly in the plugin
source tree since it needs a small init patch — only the binary lives here.

## Building

Requires Rust with `wasm32-unknown-unknown`, an LLVM/clang with wasm support
(ring's build script needs `CC`), and `wasm-bindgen-cli 0.2.x` matching
`Cargo.lock`:

```sh
cargo build --manifest-path wasm-bridge/Cargo.toml --target wasm32-unknown-unknown --release

# generate the glue (JS side is patched separately in the plugin)
wasm-bindgen wasm-bridge/target/wasm32-unknown-unknown/release/wasm_bridge.wasm \
    --out-dir out --target web --out-name wasm_bridge

# attach out/wasm_bridge_bg.wasm to a new GitHub release
gh release create vN.N.N out/wasm_bridge_bg.wasm
```

Version scheme: bump the tag whenever the bridge source or patch changes.
