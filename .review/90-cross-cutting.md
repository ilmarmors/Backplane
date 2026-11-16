# 90 Cross-cutting pass

Commit `9c6c5448a514ebcddca5a776933793e25179a796` on `main`. Done by the lead reviewer (Fable 5.2) and seven Fable reviewers who resolved the cross-unit questions. Inputs: the 61 unit files, `_verify.md` (re-traces), `_notes-phase3.md` (tool runs and scans), the seven `91-questions-*.md` files.

## 1. Re-trace of High findings from the smaller models

The rule set during the audit: a Confirmed High or Critical finding from an Opus or Sonnet unit counts as Possible until the lead re-traces it. Every such finding was re-traced at the cited lines; the log is `_verify.md`. Outcome:

- **Confirmed and kept High** (12 rows from units 39, 41, 44, 45, 48, 50, 55, 58): always-on voice with the cloud engine; the three Gerber reader bounds; the unchecked sheet path; the VRML copy blow-up; the embedded-file bomb; both browser engines without a sandbox; the settings field whitelist (two rows, one defect); the file-link open path.
- **Confirmed and lowered to Medium** (8 rows): the Boat artifact copy-back (unit 35, maintainers' pipeline only); pad rotation in the 2D reader (unit 44, a wrong picture); the full-frame capture in the KiCad pane tick (unit 53, a cost, not a wrong result); the Markdown quote recursion on Android (unit 59, needs thousands of `>`); the stale `backplane-e2e` skill (unit 56, two rows, not shipped); the stale test drivers (unit 61, two rows).
- **Refuted**: none.

Spot checks of Fable findings by the lead, all confirmed: the copy-guard marker (`native/host/src/main.rs:139`, found independently by units 02 and 03); the agent's path to the pairing token (`agents/proc.rs:299` with `scripts/bp-lock.sh:14-16`); the built-in order server address (`settings/fab.rs:13`); the bring-back of design files (`kicad/canonical.rs:284`, with the mitigation that main's copies are backed up first); the desktop upload size against the socket limit (`native/gpui/src/screens/filesui.rs:290` with `wire/mod.rs:201`, a new High from the question pass).

## 2. Cross-unit questions

The unit reviewers raised 489 questions about code outside their units (477 when the question files were cut, 12 added from the last two units). They were bucketed by the area whose code answers them and resolved against the code by seven reviewers; the files are `91-questions-*.md`, each with an answer and evidence per question.

| File | Questions | After merging | No issue | Existing finding | New finding | Open |
|---|---:|---:|---:|---:|---:|---:|
| `91-questions-a-store-wire-proc.md` | 64 | 55 | 9 | 42 | 4 | 0 |
| `91-questions-b-net.md` | 79 | 67 | 7 | 48 | 11 | 1 |
| `91-questions-c-agents.md` | 62 | 56 | 8 | 36 | 10 | 2 |
| `91-questions-d-state-session-data.md` | 69 | 57 | 14 | 35 | 8 | 0 |
| `91-questions-e-settings-bots.md` | 71 | 68 | 9 | 40 | 15 | 4 |
| `91-questions-f-kicad-files-term.md` | 73 | 60 | 7 | 39 | 10 | 4 |
| `91-questions-g-clients-guard-misc.md` | 71 | 64 | 8 | 41 | 14 | 1 |
| **Total** | **489** | **427** | **62** | **281** | **72** | **12** |

What the question pass changed:

- **New findings**: 74 rows, listed in each file's "New findings" table and merged into `findings-all.md` (Unit column `questions`). One is High: the desktop's Files upload sends a file of up to 24 MiB as one request on a socket limited to 512 KiB, so any file over about 390 KB drops the connection (`native/gpui/src/screens/filesui.rs:290`).
- **Rows corrected**:
  - Unit 24, `native/host/src/bots/mail/funnel.rs:114` (Low): its premise that nothing masks events on commit is wrong; `commit_many` masks every event (`native/host/src/hub.rs:292`). Removed from the register.
  - Unit 49 (a Sonnet unit), `native/host/src/present/window.rs:130` (Medium): the panic on a multi-byte `fold` value is not reachable. `session/dispatch.rs:70` handles `fold` only when the value starts with `0` or `1`, and any other value is refused at `present/screen.rs:674` before the thread view is built. Removed from the register.
  - Unit 14, `native/host/src/agents/locks.rs:2`: "only leases differ today" does not hold on the policy path; a heartbeat's stale whole lock record can undo a release or drop a waiter (`state/watchdog.rs:88`, a new Medium finding).
  - Unit 47: the phones' reply parser `bots::reply::of` does have the `chars=` count check. Unit 60: `/bots/<id>/screen.jpg` already answers `no-store`; only `/files/get` does not.
  - Unit 31, `native/host/src/google/mod.rs:55`: the condition "the vault opened later" cannot happen today, because a locked vault stays locked for the life of the process. Kept as Possible with that note.
  - Unit 20, `native/host/src/files/imgroutes.rs:22` (Low): `/message-image` accepts only the hub's root token, so a desktop client paired with a hub on another machine gets 401 for every image in a reply. Raised to Medium.
  - Unit 07 wrote that the always-on public door "held up". It shares the missing deadline that units 02 and 04 reported for the other listeners (`native/host/src/wire/mod.rs:326`): there is no time limit before the first byte, so one silent connection holds a permit for up to 24 hours.
  - Unit 05, `native/host/src/net/rpc.rs:348`: the duplicate of a request still in flight is covered by the actor's `recent` list; the case across a hub restart stands. `net/farview.rs:18`: replies through the tunnel are capped too (2 MiB).
  - Unit 46, `native/host/src/present/merged.rs:76`: the fallback key needs `machine.id`, not the link id, so the collision is narrower than stated.
  - Unit 61: 41 test files, not seven, call the missing `scripts/boat-job.sh`.
  - Unit 28, `native/host/src/kicad/watch.rs:291`: the "reconnect, watch, dropped again" loop holds for a phone on the Board or Schematic page; the desktop never re-sends `kicad.watch` after a reconnect (a defect of its own).
  - Unit 01: a hung writer does not leave `/healthz` green, `native/host/src/guard.rs:277` watches the writer; the built-in `turn.start` also serves the whole run when `Agents::start` fails.
- **Left open** (12 questions): they need the KiCad fork's source (whether `GetItems` on an open schematic answers other sheets' symbols, whether an item status `edited` exists, what happens to a commit left open, how the event sequence number is counted), the removed 0.14 writer of the device registry, the private fab pack on the order server, or a provider's own behaviour (the usage endpoint for setup tokens, Codex's default environment excludes).

## 3. Integration between units

### Interfaces whose two sides disagree

The full list, with both locations for each, is in section 7 (collected from the question files). The ones with user-visible effect:

- **Message size limits.** The view socket takes 512 KiB a message (`wire/mod.rs:201`). The desktop sends its whole session back with every request and never asks for segmented replies (`native/gpui/src/hub.rs:115`, `:344`), stores whole diffs in that session (`session/pages.rs:319`), and uploads files of up to 24 MiB in one request (`screens/filesui.rs:290`). A far reply is cut at 1 MiB and a tunnelled one at 2 MiB, while `viewer.pull` returns up to 6 MiB and `files.get` up to 24 MiB (`net/botnet.rs:59`, `net/farview.rs:18`, `net/farfiles.rs:16`). The hub drops a client whose outbox passes 8 MB and iOS refuses a message over 16 MiB, while plot and model frames have no byte cap (`hub.rs:450`, `kicad/watch.rs:291`).
- **401 means two things.** The hub answers 401 when it cannot write a device's `lastSeen` stamp (`net/pairing.rs:156`); all three clients read any 401 as "revoked, pair again" and stay on that screen.
- **Watches nobody renews.** `kicad.watch`, `mgmt.watch` and `kicad.checks.watch` expire or die with the socket on the hub; the desktop asks once per project, Android on each reconnect, iOS never (`native/gpui/src/screens/mgmt.rs:96`, `kicad/bompanel.rs:229`).
- **Requests the hub serves and no client sends**: `mail.*`, `bots.teach`, `kicad.retry`, `review.release`, `kicad.export10`, `editor.kicad.restart` and `close`, `/auth/known`, `zone.report`, `device.drop`; pushes no client reads: `fab.changed`.
- **The hub offer.** The hub publishes `hub.offer` with `credential: "same"` and an unchecked address taken from the client-writable setting `home.at`; the desktop and both phones send their device credential to that address without asking (`net/homemove.rs:671`, `native/gpui/src/hubs.rs:85`, `mobile/ios/Backplane/DataClient.swift:326`).
- **Phones against the desktop.** The phones sort, nest and shelve threads by their own rules instead of the hub's (`sidebar.rs` against `ThreadGroups.swift:24` and `ThreadGroups.kt`); the phone list shows mail runs the desktop hides (`data/mod.rs:461`); a plot's "none" frame is sent as binary CBOR that both apps drop (`data/mod.rs:800`); the phones hold one thread subscription while the hub serves each `subscribe` separately.
- **The wire dictionary** of the hub and the client agree today (50 keys, 61 words, same order; unit 02), but no test spans the two crates, and the two crates pin different tungstenite lines (0.24 and 0.28).

### Error propagation

- A failed commit is reported only to its caller: no log line, no degraded state, no client notice (`hub.rs:369`). 97 call sites discard the result; `commit_events` returns nothing; tools answer ok after a failed commit (`agents/review.rs:15`, `agents/mailroom.rs:64`); the turn actor carries on (`agents/mod.rs:641`).
- A spawn refused by the copy guard or safe mode surfaces as "No such file or directory" (`proc.rs:62`).
- Helper programs' stderr is discarded (browser helper, voice helper), and the voice helper prints one `error` line with exit 1 for every failure, so the hub restarts it once a second for a bad key or a missing microphone (`voice/session.rs:190`, `native/voice/src/out.rs:29`).
- The order flow reads only `error` from a failed answer while the server also sends `{"detail": …}` and nginx sends HTML (`settings/fab.rs:207`).

### Shared state and locks

- **The policy mutex** guards the one copy of the folded state. Whole views are built while holding it, including file-system walks and reading `.git/config` (`present/screen.rs:38`, `:315`); the phones' list recomputes per-row indexes under it (`data/mod.rs:95`). A panic in any reader turns the policies off for up to 60 s (`state/policy.rs:126`), and a README can panic there (`md/doc.rs:305`).
- **The bots world lock** is held across signed requests to other machines (`bots/tools.rs:130`), and **the agents' world mutex** across `git worktree add` with no time limit (`agents/mod.rs:1154`).
- **Two lock-order inversions**: mirrors lock then policy mutex in `data/mod.rs:120` against the reverse elsewhere; bots world lock then `work_lock` in `bots/swarm.rs:141` against the reverse in `agents/locks.rs:90`.
- **Requests are awaited in the socket loop** (`wire/mod.rs:228`, `data/mod.rs:780`): one slow request (an editor request can take up to 600 s, a sign-in check 12 s) blocks that client's connection, and the desktop gives up after 35 s without a frame.

### Configuration and environment

- More than 150 `BACKPLANE_*` variables are read across the hub, with no single list and no check for unknown names. Test and fault switches are compiled into release builds and read at run time (`BACKPLANE_FAULT`, `BACKPLANE_VAULT_FAULT`, `BACKPLANE_VAULT_TEST`, `BACKPLANE_KICAD_TEST_STALLS`, `BACKPLANE_TEST_*`); the units that own them report which are reachable.
- The list of variables kept from child processes (`proc.rs:54`) removes `BACKPLANE_HOME` but the same path is then handed to every agent under two other names (`agents/proc.rs:299`).
- `setting.set` refuses five keys and accepts every other from any client (`settings/set.rs:17`), which makes `home.at`, `registry.*`, `sync.*`, `machine.id`, `alert.*`, `provider.resume.*` and `fallback.*` client-writable.

### The event contract

- The project's rule is that an event's meaning never changes. `ThreadArchived` was changed in place (commit c278e6661): the fold now reads it as a manual settle and ignores `archived:false` (`state/mod.rs:220`), while the agents' world fold still sets and clears an `archived` flag that eight agent paths refuse on (`agents/world.rs:116`).
- Several writers append to the permanent log at a rate tied to clicks or timers: a `SettingSet` per selection change in KiCad (`kicad/xprobe.rs:273`), a full registry snapshot every ten minutes (`net/registry.rs:169`), the whole account inventory on every registry revision (`settings/accounts_hub.rs:21`), and an fsync for each.
- Lines imported by a home move or the 0.14 import are "past events, not news" for observers only; the followers that re-read the store cannot tell, so a carried turn end can be triaged again (`bots/mail/run.rs:124`).
- Checkpoints: `write_atomic` has no fsync and uses one temp name per process for two writers (`ckpt.rs:22`); a read error in `fold_tail` is saved as a valid checkpoint (`state/host.rs:59`).

### The same problem solved in different ways

- Signature checks: `/far/rpc` uses a replay cache, `/hub/control` and the tunnel do not (`guard.rs:261`, `net/alwayson/tunnel.rs:127`); the signature covers neither route nor direction.
- Two sheet walkers with different rules (`kicad/bompanel.rs:98`, `files/rv.rs:72`); two reply parsers (`bots::reply::of` for phones, `reply_split` for the desktop); three copies of `loopback()` in the presenter; two `layer_name` functions that differ and three nanometre readers in the KiCad edit code; four `Out` types and helper sets in the provider drivers; the main checkout's path rebuilt three ways in the scripts.
- Review comment ids are made by two generators that can coincide (`files/rv.rs:370`, `agents/review.rs:13`).
- "Who is the person": `Req.trusted` (any loopback peer, `main.rs:154`) is read as the owner in `google/mod.rs:275`, while `net/owner.rs:77` has the stricter check; `Caller` carries no device identity, and `alwayson.act`, `home.move`, `project.pick`, `kicad.install` and `editor.kicad.attach` ignore their caller.

### Authentication and authorisation, across units

- Any authenticated connection has every right: a paired phone can open a shell (`term.open`), open any file with the system opener (`file.open`), drive a sign-in shell on another machine (`signin.act`), start a home move and attach a browser endpoint. There are no per-device rights.
- An agent is not kept apart from the person: it can read the pairing token, satisfy a "the person said so" check by sending itself a message, and in an unattended mail run its permission prompts are answered "allow" unless a short deny-list matches.
- A machine link grants unscoped control by design and leaves no record of what was done over it.

## 4. Project-level health



### Tools on this machine (macOS 15, x86_64)
- No `cargo`, `rustc`, `cargo-audit`, `cargo-deny`, `nextest`, `clippy`: the Rust tests, lints and audits of native/host, native/gpui, native/voice and native/guard cannot run here.
- No `bun`: tools/browser tests and the `test/**/*.mjs|ts` drivers cannot run.
- `python3` 3.11.4 (the README asks for 3.12+), `pytest` 9.0.3, `node`, `swiftc` 6.2.4, `xcodebuild`, `gradle`; Java is 11 (CI uses 17), no ANDROID_HOME set.
- No `shellcheck`, `gitleaks`, `trufflehog`.

### Tests run
- `python3 -m pytest order/tests -p no:cacheprovider`: 24 passed, 8 failed. All 8 failures are one environment cause: `python-multipart` is not installed here ("The `python-multipart` library must be installed to use form parsing"). `cadquery-ocp` is missing too. Installed fastapi is 0.136.3, the pin is 0.120.4. Not a code failure.
- `python3 scripts/i18n-check.py`: ok, 4325 keys, every one in ja.
- `python3 scripts/i18n-android.py --check`: ok, Msg.kt is current.
- `python3 scripts/check-android-i18n.py`: ok.
- `scripts/mobile/ios-unit-tests.sh` (the CI ios job's unit step; swiftc in a temp folder): exit 0. The three catalog checks pass and all 19 suites pass: hub-list, thread-groups 17, badge-symbols 43, thread-find 21, voice-edit 25, plot-parts 27, wire-json 25, term-stream 19, vt 105, term-card 6, migration 18, tr 58, viewer-backdrop 15, browser-input 40, google-loop 21, swarm-model 23, section-menu 6, mcp-page 24, outbox 7 (XCTest).
- Android Gradle tests: not run (Java 11 here, the build needs 17; Gradle would download its distribution and dependencies and write build folders into the tree).

### CI coverage (.github/workflows)
- ci.yml jobs: author check, guard tests (3 OS), hub tests (3 OS, nextest, 2 partitions), client tests (3 OS), embedded browser lifecycle (Linux), Windows smoke, Android unit tests and catalog checks, iOS simulator build and unit tests. All gated by path filters.
- No CI job runs: native/voice tests (it is only built in release.yml), the order server's pytest, tools/browser tests on pull requests other than browser-e2e, native/step2glb.
- No clippy, rustfmt, cargo audit, cargo deny, shellcheck, coverage measurement, Dependabot or Renovate, CodeQL.
- No SECURITY.md, CONTRIBUTING.md, CHANGELOG, CODEOWNERS, issue or pull request templates.
- Third-party actions are pinned by tag, not commit SHA (dtolnay/rust-toolchain@stable, Swatinem/rust-cache@v2, rui314/setup-mold@v1, taiki-e/install-action@nextest, dorny/paths-filter@v3, oven-sh/setup-bun@v2, gradle/actions/setup-gradle@v4); release.yml uses them in a workflow that holds the release signing key.
- The Rust toolchain is `stable` with no `rust-toolchain.toml`: builds are not pinned to a compiler version.
- The hub path filter is native/host/**, skills/**, assets/**, ci.yml: a change to scripts/**, test/** or deploy/** runs no hub job.

### Secrets in the repository
- Pattern scan of tracked files (AWS, GitHub, OpenAI, Anthropic, Slack, Google, Tailscale keys, private key blocks): the only matches are test strings in mask tests (native/gpui/src/streamer.rs, native/host/src/settings/vault.rs, accounts_*.rs), a script that feeds fake keys to streamer mode, and the vendored ed25519 PEM parser.
- test/fixtures/v014-home/secrets/* and .credentials.json hold placeholders ("sk-fixture-not-real", {"fixture":true}).
- native/host/src/update/release.pub is the public release key; its README names where the private key lives (a repository secret and an offline copy).

### Script patterns (scripts/, test/, deploy/, tools/)
- No `pkill` or `killall`. One `kill -9 $(pgrep -P $HP) $HP` in test/hub/fold_scale.sh:33 (children of a captured PID, not a name).
- 68 of 153 shell scripts set `-e`.
- No `curl | sh` inside the tree other than the documented installer one-liner in the README.

### Maintainer infrastructure in a public tree
- The maintainers' real tailnet name `tail9e0cf2.ts.net` (hosts boat-hub, backplane, desktop, framework-1) appears in about 35 tracked files: tests in native/host and native/gpui, 20 scripts, 3 docs, `.backplane/intent.md`. `/home/h/` appears in 66 files, `hs-mac-mini` in 51, `h@backplane` in 34.
- One of these is shipped code: `adopt` in native/host/src/net/alwayson/mod.rs:110-116 runs in every release build and, when `~/.config/backplane/boat-hub/id` holds one hard-coded sandbox id, records the maintainers' own host `https://boat-hub.tail9e0cf2.ts.net` as this hub's always-on Boat. It does nothing on anyone else's machine, but it is a maintainer-only special case with a personal host name compiled into the product (Low).

### Dependency state (read from lockfiles and manifests; no audit tool could run)
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

## 5. Platform differences

Collected from the units: Linux is the only platform where everything exists. On macOS there is no embedded KiCad pane, no memory cap for agents, no GPU path for the 3D view, no voice recording, no auto-pause of always-on listening, and the managed KiCad fork is several versions behind (and absent on Intel Macs). On Windows the README says "experimental": Unix-only checks are skipped in CI (`scripts/ci-hub-windows-filter.sh`), the guard has no stop handling, FreeCAD part conversion uses `nice` unconditionally, file-link paths are mangled (`present/filelink.rs:27`), long git command lines break (`kicad/histsteps.rs:161`), and job handles leak when a terminal exits by itself. Linux arm64 has an installer and no managed KiCad fork.

## 6. Docs against behaviour

- `README.md`: "The hub is written in [blank] and runs on Bun"; "laws in `LAWS`, with proofs in `PROOF` checked by the compiler"; "Building needs [blank], Bun and Python 3.12+"; links to `desktop/README.md` (an Electron shell) and `docs/local-testing.md`. None of these exist. `README.jp.md` also links to two missing Windows scripts. The README offers a browser client that `AGENTS.md` says is not in the tree.
- 18 docs open with a banner that their paths name the removed tree and that the Rust location "is unverified".
- `docs/store-checkpoints.md` describes an SQLite store, held writes and a disk-full queue that the Rust hub does not have. `docs/voice.md:200` promises that audio stays on the machine. `docs/embedded-editors.md:176` says editors outlive a hub restart. `docs/work-recovery.md` describes a retry back-off and process adoption that are stubs. `docs/slim-thread-known-missing.md` is half out of date (eight keys it lists as missing are done). `docs/mobile-desktop-parity.md` lists the terminal as missing on phones, where it now exists. `docs/COPY-GUARD.md` is honest about what is not covered.
- Source comments cite docs that do not exist: `docs/PRESENTER-PORT.md`, `docs/gpui/PARITY.md`, `docs/gpui/BROWSER-EMBED.md`, `docs/LANES.md`, `docs/HUB-M1.md`, `docs/PORT.md`, `docs/SPEC.md`.
- The maintainers' tailnet name (`tail9e0cf2.ts.net`), machine names and home paths appear in about 35 tracked files (tests, 20 scripts, 3 docs). Low, for a public tree.

## 7. Interfaces where the two sides disagree (collected from the question files)

### From `91-questions-a-store-wire-proc.md`

- Imported lines: `Hub::import_lines` promises "past events, not news" (native/host/src/hub.rs:296-298) and keeps it for observers only (hub.rs:320); the followers that re-read the store on `changed` cannot tell (native/host/src/bots/mail/run.rs:124-127, native/host/src/agents/wake.rs:102-104, native/host/src/agents/tasks.rs:174-184).
- Guard status text: native/guard/src/main.rs:136 prints "no guard for this hub" when `status.json` is unreadable; deploy/install.sh.in:196 and deploy/compat/backplane-serve.sh:29 test only for "No guard is running".
- Far reads: native/host/src/hub.rs:578 keeps `git.status` and `history.refs` as reads sent to a linked machine; native/host/src/net/far.rs:56-65 lists neither, and native/host/src/net/rpc.rs:113 refuses them.
- Plot frames: the comment at native/host/src/hub.rs:50-52 calls them droppable; `push_raw` (hub.rs:459) enqueues them as frames that must arrive, and native/host/src/kicad/watch.rs:291 sends plots through it.
- `--host`: native/guard/src/run.rs:163 passes its own first and probes it (run.rs:25); native/host/src/main.rs:52 takes the last one; deploy/compat/backplane-serve.sh:17, 52 gives 0.14's `--host` to the hub only.
- `Req.trusted`: native/host/src/main.rs:154 sets it for every loopback peer; native/host/src/google/mod.rs:275 reads it as the owner's own device (the check that also excludes `tailscale serve` is native/host/src/net/owner.rs:77).
- `Caller.client`: native/host/src/agents/mod.rs:425, 497 read 0 as the hub and anything above as a person's connection; native/host/src/net/homemove.rs:368 calls `turn.start` as client 1, a number `Hub::fresh_client` gives to real connections (native/host/src/hub.rs:86, 471).
- Time zones: docs/bots.md:906-912 and the comment at native/host/src/copy.rs:143 describe a daily `tz.checked` refresh; native/host/src/hub.rs:164 is the only writer, at the log's first start.
- Health while busy: native/host/src/guard.rs:141 answers `hung` even when `folding` or `migrating` is set; native/guard/src/run.rs:234 counts that 503 as a miss, while run.rs:289 treats the same two fields as progress on a 200.
- The writer probe: native/host/src/guard.rs:277 uses `commit_many(&[])` as a liveness check; the writer treats an empty batch as a commit (fsync at native/host/src/store.rs:188, observers, presenter fold and a `changed` wake at native/host/src/hub.rs:359-365).
- Request handlers: native/host/src/api.rs:19-21 says a handler "may take its time"; native/host/src/wire/mod.rs:228 and native/host/src/data/mod.rs:780 await it in the socket loop, and the desktop client gives up after 35 s without a heartbeat (unit 02's row).
- Built-in `turn.start`: native/host/src/hub.rs:637 answers `ok` and writes `TurnQueued`; native/host/src/agents/mod.rs:321-343 starts only running or carry-marked turns, and native/host/src/commands/plan.rs:230 needs `queue.resume`.
- Windows paths: native/host/src/kicad/project.rs:35-36 builds backslash paths; native/host/src/kicad/edchecks.rs:166 splits on `/` only.
- Signed control: native/host/src/net/sign.rs:24-31 covers `<ts>.<body>`; native/host/src/guard.rs:257-262 accepts it for `/hub/control` with no replay cache and no path in the signature.
- `set_info`: native/host/src/hub.rs:196, 202 report a refusal only by returning false; about 60 publishers ignore the result, and two log it (native/host/src/term/browser.rs:149, native/host/src/term/driver.rs:102).

### From `91-questions-b-net.md`

- `Model::page` caps `limit` at 32 and returns 32 + 17 rows (native/host/src/model.rs:199-201); `home.deliver` and `has_msg` ask for 64 and take the answer for "the thread has it" (native/host/src/net/homemove.rs:230, :357).
- kedit is given the installed runtime's capabilities (native/host/src/agents/design.rs:151) while the request is checked and routed with the running KiCad's own (native/host/src/kicad/ipc.rs:496).
- `commit_design` lists files relative to the repository's top (native/host/src/kicad/canonical.rs:217, :224); `design_record_note` joins them to the project's root (native/host/src/agents/design.rs:221).
- The MCP server runs a tool to its end whatever happens to the caller (native/host/src/agents/mcp.rs:146); the lock lane frees a document's lock when the holder's turn ends and counts only turns, watches and jobs as busy (native/host/src/agents/design.rs:245, native/host/src/agents/locks.rs:181).
- `vault::put` says its caller has checked the value with `secret::bad` (native/host/src/settings/vault.rs:283); `home.secret` (native/host/src/net/homemove.rs:151) and the 0.14 import (native/host/src/import014/files.rs:149) do not.
- The home move's vault step sends every `hub.account.*.token` (native/host/src/net/homemove.rs:587) while `sync_out` names only the accounts made here (native/host/src/settings/accounts_io.rs:1395).
- The home move commits `BotDeleted` directly (native/host/src/net/homemove.rs:607); only the `bots.delete` request stops the bot's agent (native/host/src/bots/rpc.rs:257), and the routine tick does not look at a bot's state (native/host/src/bots/routines.rs:146-158).
- `/far/rpc` takes 256 KiB (native/host/src/net/rpc.rs:277, native/host/src/net/peering.rs:93); `owner.mail` sends a whole mail (native/host/src/bots/mail/owner.rs:197), `home.import` budgets by raw line length (native/host/src/net/homemove.rs:558) and the push body by characters (native/host/src/net/outbox.rs:78-81).
- A direct far reply is cut at 1 MiB (native/host/src/net/botnet.rs:59) and a tunnelled one at 2 MiB of base64 JSON (native/host/src/net/alwayson/tunnel.rs:118); `viewer.pull` returns up to 6 MiB (native/host/src/net/farview.rs:18) and `files.get` up to 24 MiB (native/host/src/net/farfiles.rs:16).
- The hub publishes `hub.offer` with `credential: "same"` and an unchecked `url` (native/host/src/net/homemove.rs:671-675); the desktop and both phones send their device credential there without asking (native/gpui/src/hubs.rs:85, mobile/ios/Backplane/DataClient.swift:326, Android per unit 37), and none asks the hub's `/auth/known` (native/host/src/net/routes.rs:20, :124).
- `home.at` keeps a copy of the link's url from the time of the move (native/host/src/net/homemove.rs:620); `peer.address` later changes only the `PeerSet` row (native/host/src/net/mesh.rs:64-74).
- The hub answers 401 when it cannot write a device's stamp (native/host/src/net/pairing.rs:156); every client reads 401 as revoked (native/gpui/src/hub.rs:86, mobile/ios/Backplane/DataClient.swift:292, mobile/android/app/src/main/java/dev/backplane/mobile/DataClient.kt:172).
- `/bots/pair/local` is documented as taking the hub token only (docs/pairing.md:144, native/host/src/net/pairflow.rs:250); `Peering::credential` takes any live device credential (native/host/src/net/peering.rs:53).
- `sign::check` accepts an empty key (native/host/src/net/sign.rs:30); `check_peer` (native/host/src/net/botnet.rs:49) and the tunnel route (native/host/src/net/alwayson/tunnel.rs:126) guard against it, `guard::own_machine` (native/host/src/guard.rs:261) does not.
- `Registry::list` tolerates `usedAt: null` (native/host/src/net/pairing.rs:303); `valid_invite` refuses the file that holds it (native/host/src/net/pairing.rs:71).
- `farsend::outcome` takes 401, 403 and 404 as the owner's final word (native/host/src/net/farsend.rs:44-52); the owning side answers 403 while its finder has not listed the sender yet (native/host/src/net/farsync.rs:14-22, :58).

### From `91-questions-c-agents.md`

- `ThreadArchived`: native/host/src/state/mod.rs:220 reads it as a manual settle and ignores `archived:false`; native/host/src/agents/world.rs:116-119 still sets and clears an `archived` flag that eight agent paths refuse on (mod.rs:670, 897, 930; work.rs:59; locks.rs:82; launch.rs:28; refs.rs:35).
- Dropped tool calls: native/host/src/term/driver.rs:337 says a parked `task_pause` ends "however this future ends (an agent stopped, its connection gone)"; native/host/src/agents/mcp.rs:146 awaits every tool call to its end and only `permit` (mcp.rs:268) watches the socket.
- Tool listing: native/host/src/term/tools.rs:2 and :9 say the agents lane lists the `task_*` tools from the JS hub's definitions; native/host/src/agents/mcp.rs:24, 158 and test/agents/fixtures/mcp-tools.json list none of them, while mcp.rs:238 dispatches them.
- `delegate_task`: the schema in test/agents/fixtures/mcp-tools.json makes `project` required for bots; native/host/src/agents/tasks.rs:24-50 never reads it, and a bot's thread has no project (native/host/src/bots/rpc.rs:210).
- Export folders: native/host/src/agents/prompt_text.rs:42 and native/host/src/settings/order.rs:253 allow a board folder named `<x>.d`; order/fabsrv/app.py:275 drops every `.d` folder from the operator's zip.
- Message size: native/host/src/session/composer.rs:86-89 says a message is at most 128 KB; native/host/src/agents/mod.rs:486 takes any text and native/host/src/data/mod.rs:749 allows a 40 MiB frame.
- Lock records: native/host/src/agents/locks.rs:2 says every transition runs under `work_lock`; native/host/src/state/watchdog.rs:88 writes whole `LockChanged` records from the commit transform.
- Lock order: native/host/src/bots/swarm.rs:141 takes `work_lock` under the bots' world lock; native/host/src/agents/locks.rs:90 and :186, design.rs:235 and work.rs:117 reach the bots' world lock (mailroom.rs:61) under `work_lock`.
- Models for agents: native/host/src/agents/host.rs:14 (a fixed Claude list) feeds `orchestrator_capabilities` (tools.rs:38), the prompt and native/host/src/settings/proposal.rs:20; the driver and the picker use `settings::models::info()`.
- ACP header secrets: native/host/src/mcpx/gen.rs:87-94 writes them for "the agent's stdin"; native/host/src/agents/proc.rs:312 masks every stdin line.
- Backplane tools and the runtime mode: native/host/src/agents/providers/pi_ext.js:12 and omp_ext.js:33 let Backplane's tools through on the ground of the hub's own rules; native/host/src/agents/mcp.rs:180-246 never reads the thread's runtime.
- Review act ids: native/host/src/files/rv.rs:370 uses `r<now>-<log length>`, native/host/src/agents/review.rs:13 uses `r<now>-<agents counter>`; native/host/src/kicad/review.rs:58 drops an add whose id exists.
- Resend: native/gpui/src/screens/unsent.rs:3 and native/host/src/data/mod.rs:334 rely on a `msg` id being taken once; native/host/src/agents/mod.rs:678 remembers ids in memory only.
- Failover stop: native/host/src/settings/driver.rs:462 and :471 send `Cmd::Stop` after the actor has already dropped the turn's end (native/host/src/agents/mod.rs:977, 1308) and can kill a turn started in between.
- Side-by-side environment: docs/SIDE-BY-SIDE.md:83 promises none of the hub's environment; native/host/src/agents/person.rs:89-92 removes only `BACKPLANE_*` and native/host/src/agents/proc.rs:295 skips the store variables for a person's turn.
- Swarm stop and document locks: native/host/src/bots/rpc.rs:71-73 releases a member's lock while native/host/src/agents/design.rs:64-68 checks the lock once per command of an edit in flight.

### From `91-questions-d-state-session-data.md`

- Pay link: docs/order.md:67 and native/host/src/settings/order.rs:7 say the log holds `order:<quote>` and the agent never sees the token; native/gpui/src/ui.rs:162 sends the shown text back and native/host/src/session/composer.rs:311 and :163 write it into the message.
- `ActivityLogged.at`: seconds for every fold (native/host/src/state/ext.rs:93, native/host/src/model.rs:168); milliseconds from native/host/src/kicad/open.rs:76 and native/host/src/plan/events.rs:17.
- Lock records: native/host/src/agents/locks.rs:2 says every transition runs under `work_lock`; native/host/src/state/watchdog.rs:88 writes whole records from `Policies::line`, committed later by native/host/src/agents/mod.rs:979 and :1310.
- Project root spelling: native/host/src/commands/fs.rs:8-9 stores `pwd -P` or `cygpath -am` output; native/host/src/data/mod.rs:485-490 compares it with `canonicalize(path)`.
- Merged project row: the comment at native/host/src/present/merged.rs:93 (the older local project names the row) against the sort at :134 (the livelier leads) and native/host/src/present/sections.rs:464 (`newThread` is the lead's id).
- `fold` action: native/host/src/session/dispatch.rs:70 accepts only a leading `0` or `1`; native/host/src/present/window.rs:127 slices whatever it is given and is safe only because native/host/src/present/screen.rs:674 refuses the rest first.
- Quiet terminals: the `q:` rule is in `TermRpc::call` (native/host/src/term/mod.rs:366); the phones' `term.write`, `term.resize` and `term.stop` (native/host/src/data/term.rs:90-101) call `Terms` directly.
- `clientId` on Entry: docs/PHONE-DATA.md:25 lists it; pages from native/host/src/data/mod.rs:239 never carry it and the push carries it only from memory (:599); mobile/android/app/src/main/java/dev/backplane/mobile/DataClient.kt:500 filters on it.
- Subscriptions: the hub sends one frame per subscription id (native/host/src/data/mod.rs:695); mobile/ios/Backplane/DataClient.swift:667-669 keeps one id per thread and does not filter pushes by `sub`.
- `push.state`: one hub's book (native/host/src/push/mod.rs:274); mobile/ios/Backplane/DataClient.swift:1103-1110 treats the answer as everything that is waiting.
- Update channels: native/host/src/data/mod.rs:519-521 serves `update.now` and the nightly channel; docs/PHONE-DATA.md:167-169 and both apps know stable and beta only.
- Webhook on a quiet failure: docs/bots.md:328-332 and native/host/src/settings/webhook.rs:50-52 (`due_end`, `end_kind`, no caller) against native/host/src/state/turnend.rs:172 (no record for that end).
- Import notes: native/host/src/import014/mod.rs:210 sends `{"k":"google","n":g}` for adopted sign-ins; native/host/src/present/onboard.rs:198 words every `google` note as "stays in 0.14".
- `skill.install`: the confirm is a composer row (native/host/src/session/composer.rs:297); the RPC takes `replace` from any caller (native/host/src/settings/skill_install.rs:32).
- Device presses: native/host/src/net/devicespage.rs:44 runs `device_act` in `spawn_blocking` for phones; native/host/src/present/screen.rs:33 and native/host/src/present/settings.rs:801 run it on the connection's task, the second under the policy lock.
- `/files/get` caching: native/host/src/files/mod.rs:84 sets no cache header (default `no-cache`, native/host/src/wire/mod.rs:63) while native/host/src/term/driver.rs:408 sends `no-store`; mobile/ios/Backplane/DataViewer.swift:76 fetches both through the shared URL cache.

### From `91-questions-e-settings-bots.md`

- Task owner: the term driver keys tasks by thread id (native/host/src/term/driver.rs:287) while the bots page and the presenter use the bot id (native/host/src/bots/api.rs:93, native/host/src/present/screen.rs:613).
- Run visibility: the sidebar and search hide mail runs and back offices (native/host/src/sidebar.rs:231, native/host/src/session/search.rs:51) while the phone list returns them (native/host/src/data/mod.rs:461).
- Voice retarget: the hub sends the old thread a view with `on:false` and keeps the session (native/host/src/voice/session.rs:178-183); both phones read that view as the session's end and stop the recorder without telling the hub (mobile/android/app/src/main/java/dev/backplane/mobile/DataVoice.kt:113, mobile/ios/Backplane/DataVoice.swift:184).
- Voice target: the phones send an empty `voice-target` to mean "Bub's chat" (DataVoice.kt:144-148, DataVoice.swift:221-225); the hub ignores an empty target (native/host/src/voice/session.rs:153).
- Hearing limit: the helper stops a `file` run at 90 s (native/voice/src/cmd.rs:20); the hub waits 300 s and reports five minutes (native/host/src/voice/session.rs:468).
- Charter grants: the permission prompt hands `perm_run` the name `mcp__backplane__<tool>` and the direct gate hands `charter_gate` the bare name (native/host/src/agents/mcp.rs:190, 193); `grant` compares `tool:<name>` exactly (native/host/src/bots/mail/charter.rs:28).
- Holds: the hub keys both providers' own login as `""` and keeps the first (native/host/src/settings/accounts_hub.rs:279-281); the desktop reads `holds[""]` for either provider (native/gpui/src/screens/accounts.rs:61, 122) and the phone rows for Claude only (native/host/src/settings/accounts_phone.rs:71).
- Vault put: `vault::put` states that its caller checked the value with `secret::bad` and refuses `hub.` names (native/host/src/settings/vault.rs:283-284); the 0.14 import and Google's adoption call it with unchecked values, and the import hands it the `hub.` names `read_other` returns (native/host/src/import014/files.rs:146-149, native/host/src/google/store.rs:74).
- Order errors: the hub reads only `error` from a failed answer (native/host/src/settings/fab.rs:207); the server also answers `{"detail": ...}` (order/fabsrv/app.py:95) and nginx answers HTML.
- Order retention: the approval says files "are kept only to make what you order" (native/host/src/settings/order.rs:270); the server keeps an unpaid quote's files 7 days and its row without end (order/fabsrv/store.py:87, order/README.md:67).
- Antigravity quota: the driver's message says Backplane does not move to another account (native/host/src/agents/providers/agy_cli.rs:126) while the failover moves the thread when the provider's text holds one of its limit phrases (native/host/src/settings/fallback.rs:74, 130-135) and the default order names antigravity.
- Telemetry notice: docs/telemetry.md:57 ties `telemetry.notice = setup` to setup's end; the hub accepts the `setup` action from any client at any time (native/host/src/settings/telemetry.rs:40).
- Build haiku: AGENTS.md allows real model turns only on temp homes with the person's go; scripts/claude-oneshot.py:58-64 runs `claude` on the default login and on the live hub's named accounts in every main build (documented in docs/MAIN-BUILD.md:30).

### From `91-questions-f-kicad-files-term.md`

- A deleted bot's browser page: native/host/src/bots/rpc.rs:259 leaves it to the term lane; native/host/src/term/driver.rs:413 says the bots lane's `/bots` handler replaces its own; neither exists, and nothing under `term/` reads BotDeleted.
- The bot space page: docs/bots.md:733 and native/gpui/src/screens/bots.rs:331 say the hub serves `GET /bots/<id>/space.html` and the phones show it; the only `/bots` route (native/host/src/term/driver.rs:386) serves pictures, and neither phone app has any code for it.
- Reply images: native/host/src/files/imgroutes.rs:22 accepts the root token only; a paired desktop client sends its device credential (native/gpui/src/hub.rs:50, native/gpui/src/screens/images.rs:63); docs/message-images.md says any pairing credential opens it.
- Terminal queries (DSR, DA): the hub answers only for chat terminals (native/host/src/term/mod.rs:126); mobile/ios/Backplane/Vt.swift:87 says the hub answers and a phone never does; the desktop answers only for the panel on show (native/gpui/src/screens/term.rs:198-202).
- The restart reset frame: the hub sends it without `state`, `g` or `o` (native/host/src/term/mod.rs:241); the desktop (native/gpui/src/screens/term.rs:193) and Android (TermStream.kt:71) keep the old state on a reset; iOS reads again (mobile/ios/Backplane/TermStream.swift:38).
- Terminal size: the hub sends a reset to every client on each resize (native/host/src/term/mod.rs:217); the client resizes back to its own panel on every frame of another size (native/gpui/src/screens/term.rs:265).
- Editor requests and the client's beat: the hub awaits a request inline for up to 600 s (native/host/src/wire/mod.rs:228, native/host/src/kicad/embed.rs:1308); the desktop client drops the session after 35 s without a frame (native/gpui/src/hub.rs:373).
- `editor.kicad.detach`, `frames` and `dialogs`: the client sends them from any pane that has a container (native/gpui/src/viewers/kicad_pane.rs:637, :660, :952-957); the hub applies them to the project's one session without a container id (native/host/src/kicad/embed.rs:1504).
- `mgmt.watch`: the hub's panels expect to be asked again within 30 minutes (native/host/src/kicad/bompanel.rs:229, native/host/src/review/policy.rs:51); the desktop asks once per project (native/gpui/src/screens/mgmt.rs:96), Android on each reconnect (DataMgmt.kt:83), iOS never.
- Browser `attach`: the hub checks the endpoint by prefix and takes it from any caller (native/host/src/term/driver.rs:128); the helper checks an exact loopback pattern (tools/browser/src/main.ts:1041); the client sends it for any hub (native/gpui/src/viewers/browser.rs:70); docs/browser-cef.md assumes one machine.
- 3D part frames: the hub sends `refs` for every instance and `at` only for instances with a box (native/host/src/kicad/pcb3d/stream.rs:311, :314); the client names parts only when the two counts match (native/gpui/src/viewers/solid.rs:295).
- Comparisons: the presenter builds them against the live file (native/host/src/present/viewer.rs:492); the plot actor treats every `diff` path as a saved version that never changes (native/host/src/kicad/watch.rs:38).
- BOM instances: the Review tab reads with an empty project name (native/host/src/files/rv.rs:84); the BOM panel with the project's name (native/host/src/kicad/bompanel.rs:150).
- The design's folder: `design_review` uses the main checkout (native/host/src/review/mod.rs:359); `order_review` uses `World::cwd`, the worktree when `kicad.main` is not on (native/host/src/settings/order.rs:60).
- `open_docs`: it answers `Unknown` for a silent KiCad (native/host/src/kicad/embed.rs:575); `open_schematic` reads that as "not open" and writes through the headless server (native/host/src/kicad/bomedit.rs:126).
- Plan approval: the hub's comment says the fork refuses it for agent clients from bp.13 batch 4 (native/host/src/kicad/ipc.rs:455); the pinned runtimes are older (native/host/src/kicad/install.rs:18-20), and `live.sock` (native/host/src/kicad/embed.rs:770) goes round the hub's own refusal.
- Hooks: every git the hub runs has hooks off, pushes included (native/host/src/files/git.rs:17); the design commit pushes at each turn's end (native/host/src/kicad/canonical.rs:246); docs/git.md:92 does not say that hooks, and so Git LFS, do not run.
- Held documents: docs/git.md:92 says a document another thread holds is left out of a turn's commit; `commit_design` can skip only the canonical board and root schematic (native/host/src/agents/design.rs:214, native/host/src/kicad/canonical.rs:223).

### From `91-questions-g-clients-guard-misc.md`

- Desktop upload size against the socket limit: native/gpui/src/screens/filesui.rs:290 and native/host/src/files/ops.rs:10 (24 MB) against native/host/src/wire/mod.rs:201 (512 KiB a message).
- A plot's `none`: the hub sends plain CBOR as a binary frame (native/host/src/data/mod.rs:800); both phones pass on a binary frame only when it is a dictionary-keyed plot (mobile/ios/Backplane/Plot.swift:466, mobile/android Plot.kt:570) and docs/PHONE-DATA.md:89 says JSON.
- 401 at the socket: the hub also answers it for a stamp it could not write (native/host/src/net/pairing.rs:156); all three clients read 401 as "revoked" and stop (native/gpui/src/hub.rs:86, mobile/ios DataClient.swift:292, mobile/android DataClient.kt:172).
- Credential transport: the hub prefers `Authorization: Bearer` (native/host/src/auth.rs:61); all three clients send `?token=` (native/gpui/src/hub.rs:211, mobile/ios Hub.swift:44, mobile/android Hub.kt:100).
- Typed settings fields: the hub draws fields named `vphrase`, `valwign`, `vjevmodel`, `ringq` (native/host/src/voice/page.rs, voice/ringpage.rs) and its own `bfield` list refuses them (native/host/src/present/settings.rs:885); the desktop has no other way to send them (native/gpui/src/screens/settings.rs:42).
- Update channels: the hub takes stable, beta and nightly (native/host/src/data/mod.rs:521); the phones offer two (mobile/android DataViews.kt:1013, mobile/ios DataViews.swift:1434).
- Solid frames: the hub bounds triangles only (native/host/src/kicad/pcb3d/stream.rs:279); iOS refuses a message over 16 MiB (mobile/ios DataClient.swift:250) and the hub's own outbox drops a client past 8 MB (native/host/src/hub.rs:450).
- Thread subscriptions: the hub keeps and serves every `subscribe` separately (native/host/src/data/mod.rs:695, :727); both phones hold one `threadSub` and ignore a frame's `sub` (mobile/android DataClient.kt:493, :541; mobile/ios DataClient.swift:462).
- Viewer tabs: the hub sends `viewer.choices`; the desktop draws a fixed rail (native/gpui/src/ui.rs:697).
- Bot space: the desktop says the page "is shown in the phone apps" (native/gpui/src/screens/bots.rs:331); neither phone app nor any hub route shows it (native/host/src/bots/tools.rs:183).
- Time zone: docs/bots.md:678 says a phone sends `zone.report` at connect; no client does and data/1 has no such method (native/host/src/settings/zone.rs:58, native/host/src/data/mod.rs:525).
- Management panels: the hub serves `mgmt.*` and `kicad.checks.*` to phones (native/host/src/data/ext.rs:284); Android draws them (DataMgmt.kt), iOS does not (mobile/ios DataViewer.swift:17).
- `CURRENT`: the guard publishes it only when an update is proven (native/guard/src/run.rs:291); the launchers start `builds/<CURRENT>/backplane-gpui` (deploy/install.sh.in:155), so after a rollback the window is of the build the guard left.
- Gate key against crate sources: the window key excludes `native/host` (scripts/dev-build.sh:116); the window crate compiles and reads two host files (native/gpui/src/main.rs:34, native/gpui/src/screens/badge.rs:135).
- Test wrappers against scripts: 41 files under test/ call `scripts/boat-job.sh`, which does not exist (scripts/ has `boat-pool.sh`); two drivers pass `--policy-bin`, which the hub refuses (native/host/src/main.rs:61).
- Guard stop against the watcher's wait: the guard may need about 20 s to stop (a 10 s probe, then the 10 s grace, native/guard/src/sys.rs:51-56); scripts/fresh.sh:45 waits 15 s and then removes the pid file.


## 8. Re-trace log

Each row: the unit, the location, the claim, the lead's verdict and what was read.

| Unit | Location | Claim | Verdict | Evidence read |
|---|---|---|---|---|
| 56 | skills/backplane-e2e/hub.sh:99 | passes `--web`, which the Rust hub rejects | Confirmed | native/host/src/main.rs:47-61: the flag match has no `--web` arm and the default arm prints usage and returns exit 2. hub.sh:99 adds `--web "$web"` when a `web` folder exists. |
| 56 | skills/backplane-e2e/hub.sh:142 | `stop` reads /proc, absent on macOS | Confirmed | hub.sh:142 reads `/proc/$p/cmdline`; on macOS the `case` never matches, the kill is skipped and line 144 removes the pid file. |
| 61 | test/hub/boat-*-job.sh | call scripts/boat-job.sh, which does not exist | Confirmed | `ls scripts/boat-job.sh`: no such file; seven test/hub/boat-*-job.sh files name it. |
| 61 | `--policy-bin` in test drivers | flag rejected by the hub | Confirmed | main.rs:61 rejects it; test/settings/boat-parity.sh:17, test/parity/boat-rust-replay.sh:14, test/parity/boat-private.sh:17, test/commands/boat-parity.sh:20 still pass it. |
| 48, 58 | native/host/src/present/settings.rs:885 | `bfield` whitelist lacks the Voice and Ring fields | Confirmed | settings.rs:885 lists 26 names and returns None for any other; voice/page.rs:159 defines field `vphrase`, voice/page.rs:142 `vjevmodel`, voice/ringpage.rs:46 `ringq`. Text typed in those fields on the desktop is dropped. |
| 35 | scripts/lib-boat.sh:94 with scripts/dev-build.sh:102-104 | a failed artifact-list read makes `bo_pull` return 0 and stale local binaries are staged as the new build | Confirmed; severity lowered to Medium (maintainers' dev pipeline, needs a failed remote read at that moment) | lib-boat.sh:92-94: the list is read with `2>/dev/null` and an empty list returns 0. dev-build.sh:12 sets only `-u`; `cpart` (line 103) is an unchecked `cp -f`, so a missing source leaves the old binary in the target folder and line 106 records "built on Boat". The sha256 list also comes from the box being checked. |
| 50 | native/host/src/present/filelink.rs:84 with native/host/src/files/misc.rs:67-76 and files/scripts/file_open.sh | a link to any existing file that is not a design or previewable file is opened with the system opener, with no type or location filter | Confirmed as a code path; High stands with its condition (a hostile or prompt-injected agent writes a launcher file and the person clicks its link) | filelink.rs:84 and :96 return `Plan::Os(path)`. misc.rs:67-76: `file.open` ignores the caller, takes any path, checks only for NUL and the copy switch, and runs file_open.sh, which runs `open "$p"` on macOS or `xdg-open` on Linux. No confinement to a project and no refusal of executables or app bundles. |
| 44 | native/host/src/kicad/gerber/rs274x.rs:135, :140, :272 with :64 | vertex counts from the file are not capped | Confirmed | Line 135: `n = g(1) as usize` then `(0..=n).map(..).collect()`. Line 140-141: `n` as u32 then `(0..n)..collect()`. Line 272 keeps `n` as u32 with only a lower bound of 3, and line 64 builds `n` points per flash. One aperture definition can ask for gigabytes. |
| 44 | rs274x.rs:245-248 with :241 | step-and-repeat counts are not capped and `nx * ny` can overflow | Confirmed | `nx`, `ny` are u32 parsed from the file with only `.max(1)`; `nx * ny` is unchecked u32 multiplication (wraps in release, panics in a debug build); line 241 pushes `nx*ny-1` copies of every object in the block. |
| 44 | rs274x.rs:94-98 | macro expressions recurse with no depth limit | Confirmed | `unary` calls itself on `-` and `+`, and `sum` on `(`, with no depth counter; a long run of `(` or `-` in one macro overflows the stack, which `catch_unwind` cannot catch. |
| 45 | native/host/src/kicad/pcb3d/models.rs:160 | an embedded zstd file in a board is unpacked with no size limit | Confirmed | Line 157-165: base64 decode, `ruzstd` `StreamingDecoder` with `read_to_end` into a Vec, then written to disk. No cap on the decoded size. |
| 45 | models.rs:489 | each VRML `USE` deep-copies what `DEF` named, so a short file grows exponentially | Confirmed | Line 489: `return self.defs.get(n).cloned()`. The depth limit of 64 on line 485 bounds parser recursion, not the size of the copied tree; a chain of definitions that each use the previous one twice doubles per step. |
| 44 | native/host/src/kicad/plotread.rs:119 with :36 | a sheet path can name a device or a huge file | Confirmed | `resolve` returns an absolute sheet file name unchanged; `version_env` reads it with `std::fs::read` and no size or file-type check. |
| 44 | native/host/src/kicad/board.rs:126 | the 2D board reader turns pads the wrong way | Confirmed; severity lowered to Medium (a wrong picture for rotated rectangular and custom pads) | board.rs:126 builds the pad frame with `rad(-a.f(2))`; the header of pcb3d/parse.rs (lines 4-7) states that this turns the other way and that a rectangle at 45 or 135 degrees shows it. |
| 39 | native/host/src/voice/session.rs:100-108 with native/voice/src/live.rs:223-241 | always-on listening can run with the OpenAI engine, against the doc's "Audio stays on this machine" | Confirmed | session.rs:100 picks the engine from the setting (`auto` falls to OpenAI when no local model is ready); line 108 adds `--always` whatever the engine and line 107 starts the helper as `Kind::Net` for the cloud engine. live.rs:223-241 does not refuse `--always` with a non-local engine; `Feed::cloud` (line 100) forwards to the cloud session when there is no local engine. docs/voice.md:200 promises "Audio stays on this machine; nothing is kept or sent before a message starts". Which segments are sent before a message starts was traced by the unit reviewer and not re-read in full. |
| 41 | tools/browser/src/main.ts:248-255 | the agents' browser is started with Chromium's sandbox off | Confirmed by the launch options and Playwright's documented default (`chromiumSandbox` defaults to false); node_modules is not installed here, so the default was not read in source | `launchPersistentContext` is called with `executablePath`, `headless`, `viewport`, `acceptDownloads` and four cosmetic args, and no `chromiumSandbox: true`. |
| 41 | tools/cef/host.cc:466, tools/cef/CMakeLists.txt:10 | the CEF host turns the sandbox off | Confirmed | host.cc:466 `st.no_sandbox = true`; CMakeLists.txt:10 `set(USE_SANDBOX OFF)`. |
| 55 | native/gpui/src/viewers/cef.rs:410, :417, :291 | the client's embedded Chromium runs without a sandbox, with a loopback DevTools port and a fixed-key cookie store | Confirmed | cef.rs:410 `no_sandbox: true`; :417 `remote_debugging_port: port` with `persist_session_cookies: true`; :291 adds the switches `use-mock-keychain` and `password-store=basic`. CEF's DevTools port has no authentication. |
| 55 | native/gpui/src/viewers/cef.rs:156-171 | every download goes to the person's Downloads folder with no prompt | Confirmed | :156 `can_download` returns true for every URL; :157-159 picks Downloads with "no dialog"; :170-171 moves a file the attached helper downloaded into Downloads as well. |
| 53 | native/gpui/src/viewers/kicad_pane.rs:679 with native_child.rs:674-683 | the pane's tick captures the whole KiCad frame only to learn whether it is mapped | Confirmed; severity lowered to Medium (a steady CPU and copy cost on the UI thread, not a wrong result) | kicad_pane.rs:329 ticks every 40 ms; :679 uses only the `mapped` flag of `nc::report`; native_child.rs:678-680 reads the map state and then, when mapped, calls `get_image` for the full window and samples it. |
| 59 | mobile/android/app/src/main/java/dev/backplane/mobile/DataMarkdown.kt:102-105 | a quote line recurses once per `>` with no depth cap | Confirmed as a code path; severity lowered to Medium (needs a line of thousands of `>`; one thread on one phone) | Lines 102-105 strip one `>` from each quoted line and call `parse(q)` again; nothing bounds the depth. |
| 02, 03 | native/host/src/main.rs:139 with native/host/src/copy.rs:150, :175 | one run with `BACKPLANE_COPY=1` marks the copied home as the hub's own, and later runs without the switch pass the copy gate | Confirmed (two Fable reviewers found it independently; re-read by the lead) | main.rs:132 runs the gate, :139 calls `copy::mark(&a.home)` with no `copy::on()` check. copy.rs:175 writes the marker when it is absent. copy.rs:150: `looks_real` returns None as soon as the marker exists, before any other check. A home copied as a whole folder carries the marker too, so it passes the gate on its first start. |
| 44, 45 | where the plot, Gerber and 3D readers run | the readers run inside the hub process | Confirmed by a Fable unit | Unit 28 (Fable) read native/host/src/kicad/watch.rs:545 and :390: sources are read inside the hub with no size, file-type or time limit; `bp-plot` is the same reader as a separate program and could host them. This is the premise of the "hostile file takes the hub down" findings of units 44 and 45. |
| 32 | native/host/src/settings/fab.rs:13 | the built-in order server is a bare IP address | Confirmed (line read by the lead) | `pub const DEFAULT_SERVER: &str = "https://2.29.106.88";` |
| 39 | native/voice/src/live.rs:239 (the helper side of the always-on finding) | what audio leaves the machine before a message starts | Confirmed by a Fable unit | Unit 40 (Fable) traced the helper: `--always` only removes the 10-minute limit; with the OpenAI engine the helper sends the dictionary at session start and then every sound above the energy threshold (300 ms before, up to 700 ms after) as it is heard, before the hub's gate sees a transcript. This closes the part of the unit 39 row that the lead had not re-read. |
| 28 | native/host/src/kicad/canonical.rs:284, :289 | bringing an older worktree's design files back compares bytes only, so a worktree that is merely behind overwrites main's newer boards | Confirmed (lead spot check), with a mitigation the row should carry | Line 284 keeps every design file whose bytes differ from main's, with no check of which side is newer. `bring_back` (line 286-292) first copies main's versions to `<main>/.backplane/backup/<stamp>-<thread>/` and then copies the worktree's files in. It is reached through the agent tool `kicad_bring_back` (agents/design.rs:272). The overwritten versions stay in the backup folder and in git history. |
| 14 | native/host/src/agents/proc.rs:299 with scripts/bp-lock.sh:14-16 | every agent is told where the hub's home is and can read the pairing token there | Confirmed (lead spot check) | proc.rs:299 sets `BACKPLANE_AGENT_OWNER` and `BACKPLANE_LOCK_HOME` to the hub's home for every agent, with the comment that `scripts/bp-lock.sh` finds its hub by it. bp-lock.sh:14-16 reads `$home/token`, the pairing token, itself. Agents run as the hub's own user with no sandbox, so an agent can read the token and call the client API as the person. |
| questions (g) | native/gpui/src/screens/filesui.rs:290 with native/host/src/wire/mod.rs:201 | the desktop's Files upload sends a file of up to 24 MiB as one request on a socket limited to 512 KiB | Confirmed (lead spot check of a new High finding from the question pass) | wire/mod.rs:201 sets `max_message_size` and `max_frame_size` to 524288. filesui.rs:290 accepts files up to `24 << 20` bytes and line 291 base64-encodes the whole file for one request, so anything over about 390 KB exceeds the limit and the socket is dropped. |
