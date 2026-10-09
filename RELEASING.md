# Releasing qrustls

## 0.23.45 provenance

- Upstream tag: `v/0.23.45`.
- Upstream commit: `2976d90fd1c2db6b518700dd101b714069cfcb17`.
- Previous qtls dependency: `22dec513c4ebdf89f113f46e100393cf18963aa6`
  (Rustls 0.23.31 plus the two patches below).
- Preserved patch: `e9f0b4ce1e09f9325b69c9a132712db549f82224`, client OCSP stapling.
- Preserved patch: `22dec513c4ebdf89f113f46e100393cf18963aa6`, TLS 1.3 session codec.

Use the `qrustls-0.23.45` branch. The original `main` branch is a 0.24 development
version and is not compatible with qtls's current backend integration.

The Cargo package is `qrustls`, version `0.23.45`; the library name remains
`rustls`. Keep dependencies aliased with `package = "qrustls"`. The fork uses
upstream's licenses. Only `qrustls` is intended for publication from this workspace.
The repository URL remains `https://github.com/genmeta/rustls`; the local directory
rename does not rename the GitHub repository.

## Validation

From this repository:

```sh
cargo test -p qrustls --lib --no-default-features --features std,ring
cargo test -p qrustls --no-default-features --features std,ring,tls12 --test api --test client_cert_verifier --test server_cert_verifier --test unbuffered
cargo test -p qrustls --doc
cargo publish -p qrustls --registry crates-io --dry-run
```

From the sibling `../dquic` repository:

```sh
cargo test -p qtls
cargo test -p qtls --no-default-features --features aws-lc-rs
cargo test -p qconnection --test components
cargo test -p qtransport --lib --tests
cargo check --workspace --all-targets
```

The qtls tests cover mutual authentication, mandatory and optional client OCSP,
server OCSP, QUIC v1/v2 keys, and stateful/stateless resumption. The upstream
API tests also exercise TLS 1.2/1.3 and QUIC handshake behavior. Tests using local
sockets require an environment that permits binding sockets.

During local preparation, `--allow-dirty` may be added to the dry run to verify
uncommitted changes. Before a real release, commit the reviewed changes and repeat
the dry run without that flag. Verify FIPS and supported cross-platform/MSRV builds
in CI; local Ring/AWS-LC tests do not establish FIPS certification.

## Publication order

1. Review the diff and CI, commit the release, and verify a clean checkout.
2. Confirm availability/ownership of the `qrustls` name on crates.io. A local
   package or successful dry run does not reserve the name or verify upload rights.
3. Repeat `cargo publish -p qrustls --registry crates-io --dry-run`.
4. Publish only after approval: `cargo publish -p qrustls --registry crates-io`.
5. Tag the release as `qrustls/v0.23.45` and push the reviewed branch and tag.
6. After crates.io serves `qrustls 0.23.45`, remove the local `path` fields from
   the three qrustls dependencies in dquic (`qtls`, `qconnection`, `qtransport`),
   retain their package aliases and exact versions, then rerun the dquic checks.
7. Prepare qtls and its dependent crates separately before removing their
   `publish = false` flags; qtls integration tests also rely on workspace fixtures.

External crypto providers compiled against upstream `rustls` have distinct types
and cannot be passed to qrustls. In particular, the optional upstream Graviola
benchmark backend is not part of the qrustls release validation.
