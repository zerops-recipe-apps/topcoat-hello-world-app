# topcoat-hello-world-app

Topcoat SSR hello world with module-based routing — baseline Rust recipe on Zerops.

## Zerops service facts

- HTTP port: `8080` (via `HOST=0.0.0.0` and `PORT=8080`)
- No sibling services — single-service SSR app with in-memory routing
- Runtime base: `rust@stable` on Ubuntu

## Zerops dev

`setup: dev` idles on `zsc noop --silent`; the agent starts the dev server.

- Dev command: `topcoat dev` (requires `topcoat-cli` installed in `prepareCommands`)
- Alternative: `cargo run`
- In-container rebuild without deploy: `cargo build --release`

**All platform operations (start/stop/status/logs of the dev server, deploy, env / scaling / storage / domains) go through the Zerops development workflow via `zcp` MCP tools. Don't shell out to `zcli`.**

## Notes

- Topcoat defaults to `127.0.0.1:3000`; Zerops sets `HOST` and `PORT` so the L7 balancer can reach the app.
- `CARGO_HOME` is redirected to `./.cargo` in the build so Zerops can cache the registry between builds; dev run overrides `CARGO_HOME=/var/www/.cargo` for interactive SSH sessions.
- Dev build uses `cargo fetch` only — the first `topcoat dev` or `cargo run` compiles deps inside the container.
