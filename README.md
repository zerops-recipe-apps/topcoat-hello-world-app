# Topcoat Hello World on Zerops

<!-- #ZEROPS_EXTRACT_START:intro# -->
Minimal [Topcoat](https://github.com/kriszerops/topcoat) SSR app deployed on Zerops. Server-rendered "Hello, World!" with module-based routing — no database.
<!-- #ZEROPS_EXTRACT_END:intro# -->

## Quick start (local)

```bash
cargo install topcoat-cli --locked
topcoat dev
```

Open http://127.0.0.1:3000 — or set `HOST=0.0.0.0 PORT=8080 topcoat dev` to match Zerops.

<!-- #ZEROPS_EXTRACT_START:integration-guide# -->
## Zerops integration

### Build pipeline

| Phase | What happens |
|-------|----------------|
| **prod** | `cargo build --release --locked` → deploy `./target/release/topcoat-hello-world` |
| **dev** | `cargo fetch` + install `topcoat-cli`; full source tree deployed for SSH dev |

### Runtime

Topcoat binds via `HOST` and `PORT` environment variables. Zerops sets:

```yaml
HOST: "0.0.0.0"
PORT: "8080"
```

Default Topcoat bind is `127.0.0.1:3000` — without these overrides the L7 balancer cannot reach the app.

### Caching

`CARGO_HOME: ./.cargo` keeps the registry inside the project tree so Zerops caches `.cargo/registry` and `target` between builds.

### Health check

Readiness probe: `GET /` on port 8080. Topcoat serves the home page at `/` via `#[page("/")]`.

### Development on Zerops

SSH into the dev container and run:

```bash
topcoat dev
```

The dev setup installs `topcoat-cli` during `prepareCommands` and deploys source with `deployFiles: [./]`.

### Simple-mode alternative

For quick experiments you can set `deployFiles: [.]` in prod to keep the full source tree (including `target/`) across redeploys. The prod setup in this repo deploys the release binary only for a smaller runtime footprint.

### Adding a database

When the app outgrows this minimal example, add a `db` service to import.yaml (`postgresql:single@17`, `priority: 10`) and wire connection env vars in `zerops.yaml` — see the [Rust Hello World](https://github.com/zeropsio/recipes/tree/main/rust-hello-world) recipe.
<!-- #ZEROPS_EXTRACT_END:integration-guide# -->

## References

- [Topcoat getting started](https://github.com/kriszerops/topcoat/blob/main/crates/topcoat/docs/getting_started.md)
- [Zerops Rust build pipeline](https://docs.zerops.io/rust/how-to/build-pipeline)
- [zerops.yaml specification](https://docs.zerops.io/zerops-yaml/specification)
