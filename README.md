# Topcoat Hello World Recipe App

<!-- #ZEROPS_EXTRACT_START:intro# -->
Minimal [Topcoat](https://github.com/kriszerops/topcoat) SSR app deployed on Zerops. Server-rendered "Hello, World!" with module-based routing — no database.
Used within [Topcoat Hello World recipe](https://app.zerops.io/recipes/topcoat-hello-world) for [Zerops](https://zerops.io) platform.
<!-- #ZEROPS_EXTRACT_END:intro# -->

⬇️ **Full recipe page and deploy with one-click**

[![Deploy on Zerops](https://github.com/zeropsio/recipe-shared-assets/blob/main/deploy-button/light/deploy-button.svg)](https://app.zerops.io/recipes/topcoat-hello-world?environment=small-production)

![rust cover](https://github.com/zeropsio/recipe-shared-assets/blob/main/covers/svg/cover-rust.svg)

## Integration Guide

<!-- #ZEROPS_EXTRACT_START:integration-guide# -->

### 1. Adding `zerops.yaml`
The main application configuration file you place at the root of your repository, it tells Zerops how to build, deploy and run your application.

```yaml
zerops:
  # Production setup — compile optimized release binary and run on port 8080.
  # Topcoat defaults to 127.0.0.1:3000; HOST/PORT override that for Zerops routing.
  - setup: prod
    build:
      base: rust@stable
      os: ubuntu

      # Keep cargo registry under the project tree so Zerops can cache it.
      envVariables:
        CARGO_HOME: ./.cargo

      buildCommands:
        # --locked validates Cargo.lock — prevents surprise dependency updates in prod.
        - cargo build --release --locked

      deployFiles:
        # Deploy release binary only (smaller artifact than deployFiles: [.])
        - ./target/release/topcoat-hello-world

      cache:
        - .cargo/registry
        - target

    deploy:
      readinessCheck:
        httpGet:
          port: 8080
          path: /

    run:
      base: rust@stable
      os: ubuntu

      ports:
        - port: 8080
          httpSupport: true

      envVariables:
        HOST: "0.0.0.0"
        PORT: "8080"

      start: ./target/release/topcoat-hello-world

  # Development setup — deploy full source for SSH / topcoat dev workflow.
  # Developer runs 'topcoat dev' manually; Zerops does not start the app.
  - setup: dev
    build:
      base: rust@stable
      os: ubuntu

      envVariables:
        CARGO_HOME: ./.cargo

      prepareCommands:
        # Topcoat CLI provides 'topcoat dev' — watch mode with asset bundling.
        - cargo install topcoat-cli --locked

      buildCommands:
        # Fetch dependencies only — developer compiles on demand via SSH.
        - cargo fetch

      deployFiles:
        - ./

      cache:
        - .cargo/registry

    run:
      base: rust@stable
      os: ubuntu

      ports:
        - port: 8080
          httpSupport: true

      envVariables:
        CARGO_HOME: /var/www/.cargo
        HOST: "0.0.0.0"
        PORT: "8080"

      # No app started — connect via SSH and run 'topcoat dev' or 'cargo run'
      start: zsc noop --silent
```
<!-- #ZEROPS_EXTRACT_END:integration-guide# -->


<!-- #ZEROPS_EXTRACT_START:knowledge-base# -->
### Gotchas

- **Topcoat binds to `127.0.0.1:3000` by default** — without `HOST=0.0.0.0` and `PORT=8080`, Zerops readiness checks and the L7 balancer cannot reach the app.
- **Use `topcoat dev` in dev containers** — install `topcoat-cli` in `prepareCommands`; it provides watch mode and asset bundling that plain `cargo run` does not.
- **Build container has no `pkg-config` / `libssl-dev`** — crates defaulting to `native-tls` (`reqwest`, `hyper-tls`, …) fail with `Could not find directory of OpenSSL installation`. Use rustls-tls instead: `reqwest = { version = "0.12", default-features = false, features = ["json", "rustls-tls"] }`. Same pattern for any HTTP client crate.
<!-- #ZEROPS_EXTRACT_END:knowledge-base# -->
