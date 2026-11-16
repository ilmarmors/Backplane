# Working notes for the cross-cutting pass (merged into 90-cross-cutting.md at the end)

## Tools on this machine (macOS 15, x86_64)
- No `cargo`, `rustc`, `cargo-audit`, `cargo-deny`, `nextest`, `clippy`: the Rust tests, lints and audits of native/host, native/gpui, native/voice and native/guard cannot run here.
- No `bun`: tools/browser tests and the `test/**/*.mjs|ts` drivers cannot run.
- `python3` 3.11.4 (the README asks for 3.12+), `pytest` 9.0.3, `node`, `swiftc` 6.2.4, `xcodebuild`, `gradle`; Java is 11 (CI uses 17), no ANDROID_HOME set.
- No `shellcheck`, `gitleaks`, `trufflehog`.

## Tests run
- `python3 -m pytest order/tests -p no:cacheprovider`: 24 passed, 8 failed. All 8 failures are one environment cause: `python-multipart` is not installed here ("The `python-multipart` library must be installed to use form parsing"). `cadquery-ocp` is missing too. Installed fastapi is 0.136.3, the pin is 0.120.4. Not a code failure.
- `python3 scripts/i18n-check.py`: ok, 4325 keys, every one in ja.
- `python3 scripts/i18n-android.py --check`: ok, Msg.kt is current.
- `python3 scripts/check-android-i18n.py`: ok.
- `scripts/mobile/ios-unit-tests.sh` (the CI ios job's unit step; swiftc in a temp folder): exit 0. The three catalog checks pass and all 19 suites pass: hub-list, thread-groups 17, badge-symbols 43, thread-find 21, voice-edit 25, plot-parts 27, wire-json 25, term-stream 19, vt 105, term-card 6, migration 18, tr 58, viewer-backdrop 15, browser-input 40, google-loop 21, swarm-model 23, section-menu 6, mcp-page 24, outbox 7 (XCTest).
- Android Gradle tests: not run (Java 11 here, the build needs 17; Gradle would download its distribution and dependencies and write build folders into the tree).

## CI coverage (.github/workflows)
- ci.yml jobs: author check, guard tests (3 OS), hub tests (3 OS, nextest, 2 partitions), client tests (3 OS), embedded browser lifecycle (Linux), Windows smoke, Android unit tests and catalog checks, iOS simulator build and unit tests. All gated by path filters.
- No CI job runs: native/voice tests (it is only built in release.yml), the order server's pytest, tools/browser tests on pull requests other than browser-e2e, native/step2glb.
- No clippy, rustfmt, cargo audit, cargo deny, shellcheck, coverage measurement, Dependabot or Renovate, CodeQL.
- No SECURITY.md, CONTRIBUTING.md, CHANGELOG, CODEOWNERS, issue or pull request templates.
- Third-party actions are pinned by tag, not commit SHA (dtolnay/rust-toolchain@stable, Swatinem/rust-cache@v2, rui314/setup-mold@v1, taiki-e/install-action@nextest, dorny/paths-filter@v3, oven-sh/setup-bun@v2, gradle/actions/setup-gradle@v4); release.yml uses them in a workflow that holds the release signing key.
- The Rust toolchain is `stable` with no `rust-toolchain.toml`: builds are not pinned to a compiler version.
- The hub path filter is native/host/**, skills/**, assets/**, ci.yml: a change to scripts/**, test/** or deploy/** runs no hub job.

## Secrets in the repository
- Pattern scan of tracked files (AWS, GitHub, OpenAI, Anthropic, Slack, Google, Tailscale keys, private key blocks): the only matches are test strings in mask tests (native/gpui/src/streamer.rs, native/host/src/settings/vault.rs, accounts_*.rs), a script that feeds fake keys to streamer mode, and the vendored ed25519 PEM parser.
- test/fixtures/v014-home/secrets/* and .credentials.json hold placeholders ("sk-fixture-not-real", {"fixture":true}).
- native/host/src/update/release.pub is the public release key; its README names where the private key lives (a repository secret and an offline copy).

## Docs drift seen while mapping
- README "How it works": "The hub is written in  and runs on Bun. Important rules are stated as laws in [`LAWS`], with proofs in [`PROOF`] checked by the compiler." The hub is Rust; LAWS and PROOF are gone.
- README "Contributing": "Building needs , Bun and Python 3.12+", links to docs/local-testing.md and desktop/README.md (Electron shell). Check that those files exist.
- README says desktop, web and phones; AGENTS.md says there is no web client in this tree.
- 18 docs carry the banner "Paths under src/ ... name the Bend JS tree, removed from this checkout ... Where the current Rust code for each is, is unverified".
- docs/store-checkpoints.md describes an SQLite store.db; the hub has no SQLite dependency.
- docs/mobile-desktop-parity.md lists Terminal as a gap on both phones, but TermViews.kt and TermViews.swift exist: verify.

## Dead links and references
- README.md links to `desktop/README.md` and `docs/local-testing.md`: neither exists. README.jp.md also links to `scripts/install-windows-providers.ps1` and `scripts/setup-windows.ps1`: neither exists.
- Docs and source comments point at docs that do not exist: docs/gpui/PARITY.md, docs/PRESENTER-PORT.md (named in native/host/src/present/mod.rs), docs/gpui/BROWSER-EMBED.md, docs/LANES.md, docs/HUB-M1.md, docs/PORT.md, docs/SPEC.md, docs/backplane/floorplanner.md, docs/backplane/coedit.md, docs/user/usage.md.

## Script patterns (scripts/, test/, deploy/, tools/)
- No `pkill` or `killall`. One `kill -9 $(pgrep -P $HP) $HP` in test/hub/fold_scale.sh:33 (children of a captured PID, not a name).
- 68 of 153 shell scripts set `-e`.
- No `curl | sh` inside the tree other than the documented installer one-liner in the README.

## Maintainer infrastructure in a public tree
- The maintainers' real tailnet name `tail9e0cf2.ts.net` (hosts boat-hub, backplane, desktop, framework-1) appears in about 35 tracked files: tests in native/host and native/gpui, 20 scripts, 3 docs, `.backplane/intent.md`. `/home/h/` appears in 66 files, `hs-mac-mini` in 51, `h@backplane` in 34.
- One of these is shipped code: `adopt` in native/host/src/net/alwayson/mod.rs:110-116 runs in every release build and, when `~/.config/backplane/boat-hub/id` holds one hard-coded sandbox id, records the maintainers' own host `https://boat-hub.tail9e0cf2.ts.net` as this hub's always-on Boat. It does nothing on anyone else's machine, but it is a maintainer-only special case with a personal host name compiled into the product (Low).

## Dependency state (read from lockfiles and manifests; no audit tool could run)
- `cargo audit` and `cargo deny` are not installed and there is no Rust toolchain here, so no advisory database was consulted. The versions below are what the lockfiles pin.
- Hub (native/host/Cargo.lock, 106 packages): tokio 1.54.1, tokio-tungstenite and tungstenite 0.24.0, httparse 1.10.1, prost 0.14.4, aes-gcm 0.10.3, sha2 0.10.9, zip 2.6.1, flate2 1.1.11, ruzstd 0.8.3, rand 0.8.8, getrandom 0.2.17, pulldown-cmark 0.13.6, vendored ed25519-compact 2.6.0. A small tree with no TLS stack of its own (outbound HTTP goes through `curl` child processes).
- Client (native/gpui/Cargo.lock, 751 packages): gpui 0.2.2 (vendored, one build patch), cef 154.5.0+154.0.34 pinned exactly, rustls 0.23.46, tokio-tungstenite 0.28.0, image 0.25.10, resvg 0.45.1. Several crates in two or three versions (rand 0.8, 0.9 and 0.10; ttf-parser three versions; png, fontdb, ashpd and ureq two each). Transitive crates that their authors have declared unmaintained or discontinued are in the tree: `instant` 0.1.13, `paste` 1.0.15, `async-std` 1.13.2 (all arrive through gpui).
- The hub and the client pin different tungstenite lines (0.24 and 0.28) for the two ends of one socket.
- The embedded Chromium is pinned to one CEF build (154) and runs with its sandbox off (unit 55), so a browser security fix reaches people only with a client release.
- Voice helper (112 packages): ureq 3.5.1, rustls 0.23.46, tar 0.4.46, bzip2 0.6.1 (pure Rust backend), libloading 0.8.9.
- Guard (22 packages): serde_json, sha2, libc only.
- No git or path dependencies other than the vendored crates.
- Order server: fastapi 0.120.4, uvicorn 0.38.0, python-multipart 0.0.32, httpx 0.28.1, cadquery-ocp 8.0.1.1.0, pytest 9.1.1, pinned exactly but with no hashes and no lock for transitive packages (starlette, pydantic, anyio float).
- Browser tool: playwright-core 1.63.0, Bun 1.4.4 (deploy/bun-version).
- Android: Android Gradle plugin 8.7.3, Kotlin 2.0.21, compileSdk and targetSdk 35, minSdk 26, okhttp 4.12.0, zxing core 3.5.3. The Gradle wrapper pins its distribution by sha256.
- Nothing updates dependencies automatically (no Dependabot or Renovate) and nothing audits them in CI.
