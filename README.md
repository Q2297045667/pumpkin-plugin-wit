# pumpkin-plugin-wit

WebAssembly Interface Types (WIT) definitions for the Pumpkin plugin API (v0.1). These .wit files describe the host <-> plugin interface used by the Pumpkin Minecraft server runtime: the plugin world (exports a plugin implements), server resources, common types, events, players, text components, packets, and other domain types used by plugins.

## What this is
A collection of WIT interface definitions that specify how Pumpkin plugins interact with the host server. Plugin authors use these definitions to generate language bindings, implement the required exports (init-plugin, on-load, on-unload, handle-event, handle-command, handle-task, and metadata), and compile plugins to WebAssembly modules consumable by the Pumpkin host.

### Stack
- **Language(s):** WIT (WebAssembly Interface Types), intended targets: Rust / TypeScript / Go / other languages with wit-bindgen support
- **Framework / runtime:** WebAssembly modules (WIT-based host imports/exports) running inside the Pumpkin server host
- **Notable tools:** wit-bindgen (or equivalent interface-binding generator), standard wasm toolchains (rustc / cargo for wasm32 targets, or language-specific toolchains for producing .wasm)

## How it's organized
Top-level (important files only):

v0.1/
  plugin.wit         — main `world plugin` (required exports & host imports)
  common.wit         — shared types (positions, colors, enums)
  server.wit         — server resource and global APIs (get-player, create-world, sys-info, etc.)
  metadata.wit       — plugin metadata interface (name, version, authors, dependencies, permissions)
  player.wit         — player resource and actions
  event.wit          — event types and event shapes
  text.wit           — text-component and formatting types
  entity.wit         — entity types and entity-related APIs
  sounds.wit         — sound definitions / identifiers
  java-packets.wit   — Java-specific packet types
  bedrock-packets.wit— Bedrock-specific packet types
  ...                — other domain-specific interface files (biomes.wit, gui.wit, scheduler.wit, etc.)

How it fits together:
- `v0.1/plugin.wit` is the entrypoint describing what a plugin must export and what imports a host will provide.
- `common.wit` provides shared primitives used across other interfaces (positions, enums).
- `server.wit` and the other domain files describe resources the host exposes to plugins (server instance, world, player, events).
- Plugin authors generate bindings from these .wit files, implement the required exported functions and resources, compile to .wasm, and register that WebAssembly module with their Pumpkin server.

## How to use (authoring a plugin)
These WIT files are interface descriptions — they are not runnable by themselves. Typical author workflow:

1. Generate language bindings from the WIT files (use wit-bindgen or your preferred WIT tool).
2. Implement the plugin exports defined in `v0.1/plugin.wit`:
   - `init-plugin()`
   - `on-load(context)` -> result<_, string>
   - `on-unload(context)` -> result<_, string>
   - `metadata` (returning the shape from `metadata.wit`)
   - `handle-event(event-id, server, event)` -> event
   - `handle-command(command-id, sender, server, args)` -> result<s32, command-error>
   - `handle-task(handler-id, server)`
3. Build your code into a WebAssembly module (.wasm) using your language toolchain targeting the appropriate wasm target the Pumpkin host expects (wasm32-wasi or wasm32-unknown-unknown depending on host).
4. Drop the produced .wasm plugin into the Pumpkin server's plugins folder (or follow server docs to register plugins) and start/restart the server.

Example (illustrative — commands depend on chosen language/tooling):
```bash
# Generate bindings (example)
wit-bindgen generate --language rust ./v0.1/plugin.wit --out-dir bindings

# Implement plugin in Rust, then build:
cargo build --target wasm32-unknown-unknown --release

# Copy produced .wasm to Pumpkin server plugins directory
cp target/wasm32-unknown-unknown/release/my_plugin.wasm /path/to/pumpkin/plugins/
# Restart Pumpkin server (server-specific command)
```

Note: The exact binding generation and build commands vary by language and by the specific wit-bindgen toolchain you choose. Check your language's wit-bindgen integration for exact invocation.

## Key interfaces and symbols
- `v0.1/plugin.wit` — package: `pumpkin:plugin@0.1.0`, world `plugin` (the required plugin exports and host imports)
- `init-plugin`, `on-load`, `on-unload`, `metadata`, `handle-event`, `handle-command`, `handle-task` — plugin entrypoints to implement
- `metadata.wit` — `plugin-metadata` record: name, version, authors, description, dependencies, permissions
- `common.wit` — `block-pos`, `position`, `game-mode`, `locale`, color types, etc.
- `server.wit` — `resource server` with `get-player-by-name`, `broadcast`, `create-world`, `get-recipe-manager`, etc.

## Examples & quick reference
- Implementations must return metadata shaped like `metadata.plugin-metadata` (see `v0.1/metadata.wit`).
- Event flow: Pumpkin host calls `handle-event` (plugin must inspect the `event` record and return an `event` or handle it side-effectfully using server imports).
- Command flow: host calls `handle-command` with `command-id` and `consumed-args`; plugin returns a result or a command error.

## Contributing
- This repo contains WIT interface specifications. To contribute:
  - Open a PR with a clear rationale for interface changes (breaking changes must be versioned).
  - If you add or change a type, update related interface files and include examples of language bindings where possible.
  - Add a CHANGELOG entry for versioned changes.

## License
No LICENSE file detected in the repository. Add a LICENSE if you want to make the copyright/usage terms explicit.

## Try asking
- "How should a plugin implement and return `metadata.plugin-metadata` from v0.1/metadata.wit — can you show a minimal Rust example?"
- "What's the expected WASM target and host ABI for Pumpkin? Should plugins target wasm32-wasi or wasm32-unknown-unknown?"
- "Can you show a minimal plugin that registers a command in `on-load` and handles it in `handle-command` using the types in v0.1/command.wit and v0.1/plugin.wit?"
