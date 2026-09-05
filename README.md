# VencordDashBeamWasm

Release distribution for the wasm bridge used by the **DashBeam** plugin in
[Loukious' Vencord fork](https://github.com/Loukious/Vencord).

The bridge is built from [dashbeam](https://github.com/tonyantony300/dashbeam)'s
`wasm-bridge` crate (iroh P2P file transfer compiled to `wasm32-unknown-unknown`),
with a local patch:

- `receive_file` runs as a cancellable task (`n0_future::task::spawn`) with its
  `AbortHandle` stored in a static slot
- a new `cancel_receive()` export aborts the in-flight download

## Releases

Each release tags a build; the asset `wasm_bridge_bg.wasm` is what the plugin's
main process auto-downloads (versioned cache in `DATA_DIR/plugins/DashBeam`).
The JS glue (`wasm_bridge.js` / `.d.ts`) is vendored directly in the plugin
source tree since it needs a small init patch — only the binary lives here.

## Building

Requires Rust with `wasm32-unknown-unknown`, LLVM/clang (ring's build script),
and `wasm-bindgen-cli 0.2.x` matching the crate lockfile:

```sh
# 1. clone dashbeam, apply the cancel_receive patch to wasm-bridge/src/lib.rs
export CC='C:/Program Files/LLVM/bin/clang.exe'   # Windows; needed for ring
cargo build --manifest-path wasm-bridge/Cargo.toml --target wasm32-unknown-unknown --release

# 2. generate the glue (JS side is patched separately in the plugin)
wasm-bindgen wasm-bridge/target/wasm32-unknown-unknown/release/wasm_bridge.wasm \
    --out-dir out --target web --out-name wasm_bridge

# 3. attach out/wasm_bridge_bg.wasm to a new GitHub release here
```

Version scheme: bump the tag whenever the bridge source or patch changes.
