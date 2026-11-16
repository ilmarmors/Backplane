# 00 Map

Audit of the whole tracked tree (`git ls-files`), not a diff.

- Branch: `main`
- Commit: `9c6c5448a514ebcddca5a776933793e25179a796`
- Upstream: `origin/main` (git@github.com:BackplaneWorks/Backplane.git), 0 ahead, 0 behind after `git fetch --all` on 2026-11-16
- Tracked files: 3,516
- History: 7,244 commits between 2026-09-22 and 2026-11-16 (eight weeks; 3,151 commits in ISO week 45 alone). Almost all by `i2cjak <build@amemb.com>` and `jlcjak`; the commit style and volume show agent-written lanes landing through `scripts/land.sh`.

## What the project is

Backplane is an open-source (MIT) workspace for agentic hardware development. A person runs one **hub** per machine. The hub starts and supervises coding-agent CLIs (Claude, Codex, Grok, OpenCode, Muse, Antigravity, Pi, Oh My Pi), lets them edit KiCad boards and schematics through a KiCad fork's IPC API and FreeCAD scripts, and records everything in an append-only event log. A GPUI desktop client and native iOS and Android apps connect to the hub and show threads, board, schematic, 3D and mechanical viewers, design history and review.

Users: electrical and hardware engineers who run agents on their own machines, their phones as remote clients, and contributors (the tree is public and takes pull requests).

The tree was ported in the last weeks from a JavaScript hub written in a language called Bend (`src/**/*.bend`, `LAWS.bend`, `PROOF.bend`, tag `main-bend-final`) to Rust. The old tree is gone from this checkout, but the README, 18 docs with a "removed from this checkout" banner, and many source comments still point at it.

## Directory structure (tracked files, lines)

| Lines | Files | Path | Note |
|---:|---:|---|---|
| 148,815 | 300 | docs/media | images (excluded) |
| 145,017 | 556 | native/host | the Rust hub; `src/` is 138,362 lines and 11.3 MB |
| 140,436 | 293 | native/gpui | desktop client; `src/` is 37,262 lines and 3.1 MB, `vendor/` is 95,000 lines (excluded) |
| 87,954 | 114 | test/files | fixtures (excluded) |
| 38,209 | 114 | mobile/android | Kotlin/Compose app; `Msg.kt` (965 KB) is a generated catalog |
| 27,284 | 112 | mobile/ios | SwiftUI app; `Shared/TrCatalog.swift` (624 KB) is a generated catalog |
| 25,717 | 7 | assets/fonts | fonts (excluded) |
| 21,289 | 47 | native/voice | voice helper; `src/` is 5,192 lines, the rest is a vendored C header, VAD files and tests |
| 15,525 | 71 | test/kicad_embed | embedded-KiCad e2e drivers (Python) |
| 15,229 | 125 | docs (top level) | feature and process docs |
| 11,744 | 146 | test/e2e | e2e drivers |
| 9,312 | 3 | i18n | message catalogs (en, ja) |
| 9,161 | 202 | scripts | build, land, watch, device and rig scripts |
| 6,645 | 71 | deploy/kistack | KiStack skills pinned from another repository (excluded) |
| 4,192 | 34 | tools/browser | headless Chromium helper (Bun/TypeScript) |
| 3,736 | 27 | order | order server `fabsrv` (Python/FastAPI) and its deploy |
| 3,676 | 3 | assets/web | `style.css` (the colour source), favicons |
| 1,956 | 12 | native/guard | `backplane`, the hub's guard (Rust, shares no code with the hub) |
| 1,884 | 5 | deploy/skills | the `sendcutsend` skill |
| 1,836 | 19 | tools/cef | CEF host and tests |
| 1,674 | 9 | .github/workflows | CI, release, nightly, e2e |
| 1,040 | 6 | skills/backplane-e2e | agent skill |
| 355 | 2 | native/step2glb | STEP mesher (C++ on OpenCascade) |
| other | | test/* (60 small dirs), deploy/licenses, docs/* subfolders | |

Hub `src/` by module (KB of source): kicad 1,973; agents 1,413; settings 1,154; present 1,117; bots 1,025; net 828; root files 446; voice 288; commands 286; session 272; files 270; term 266; state 257; data 253; fab 239; mcpx 200; google 148; review 122; import014 89; md 79; update 74; plat 73; plan 48; push 46; wire 45; bin 23.

Lines are long in this tree (average 82 characters, 5,732 hub lines over 200 characters, the longest 5,684), so review units below are sized by bytes: 1,000 to 2,500 ordinary lines is about 80 to 250 KB.

## Hotspots (commits touching a file, whole history)

The top of the churn list is the removed Bend tree (`PROOF.bend` 648, `LAWS.bend` 620, `src/server/server.bend` 455, `src/core/client.bend` 427, `src/web/view.bend` 384). Among files that still exist:

| Commits | File |
|---:|---|
| 223 | AGENTS.md |
| 216 | native/gpui/src/ui.rs |
| 205 / 203 | i18n/en.msg, i18n/ja.msg |
| 151 | native/gpui/src/main.rs |
| 108 | mobile/android/.../Ui.kt (since removed), 98 Screen.kt |
| 103 | native/host/src/agents/mod.rs |
| 101 | native/host/src/hub.rs |
| 101 | mobile/ios/Backplane/Screen.swift |
| 96 | native/host/src/agents/tests.rs |
| 95 | native/gpui/src/screens/mod.rs |
| 92 | native/host/src/present/screen.rs |
| 85 / 84 | mobile/ios DataViews.swift, Views.swift |
| 83 | native/host/src/data/mod.rs, .github/workflows/ci.yml, docs/bots.md |
| 82 / 80 | native/gpui/src/script.rs, hub.rs |
| 77 | native/host/src/main.rs |
| 76 | native/host/src/state/policy.rs |
| 75 | native/gpui/src/screens/viewer.rs |
| 67 | native/host/src/present/thread.rs, native/gpui/src/viewers/kicad_pane.rs |
| 66 | native/host/src/present/settings.rs |
| 64 | native/host/src/kicad/embed.rs |
| 61 | .github/workflows/release.yml |
| 59 | native/host/src/session/mod.rs, mobile/android DataClient.kt, 58 iOS DataClient.swift |

## Architecture

Components and entry points:

- **Hub** `native/host` (`backplane-hub`, `src/main.rs`). One per machine. HTTP and an authenticated WebSocket (`wire/`), hand-rolled on `httparse`, `tokio` and `tokio-tungstenite`. Two streams over the socket: the view stream for the GPUI client (the presenter in `present/` builds screen models, `delta.rs` sends keyed deltas, CBOR with append-only dictionaries in `wire_dict.rs`) and the phone protocol `data/1` (`data/`, plain data, JSON or CBOR). Extension points are `RpcHandler` and `HttpHandler` registered on `Arc<Hub>` (`api.rs`).
- **Event store** `store.rs`: `events.jsonl`, one JSON event per line, single writer, group commit with one fsync (`Hub::commit_many`). `model.rs` and `state/` fold the log into the read model and policies. `ckpt.rs` saves start-up checkpoints of those folds, each trusted only by a build whose fold sources hash the same (`build.rs`, `fold_inputs.rs`).
- **Agents** `agents/`: one actor per thread owns that thread's agent process. Providers in `agents/providers/` (Claude and Codex directly, ACP-based ones, Antigravity, Muse, OpenCode, Pi, Oh My Pi). The hub gives each agent a Backplane MCP server on loopback (`agents/mcp.rs`). Design-file locks in `agents/design.rs` and `agents/locks.rs`.
- **Process spawning** `proc.rs`: the one place a child is made, each with a `Kind`. `copy.rs` is the copy guard (`BACKPLANE_COPY=1`): a hub on a copy of a real log starts no network process, terminal, browser or unsolicited turn. `memscope.rs` puts agent processes in systemd scopes on Linux.
- **Net** `net/`: pairing (invitations, device credentials, QR), the tailnet listener, far threads and peers (one owner's machines talk to each other, signed requests), always-on hubs on Boat sandboxes, home moves.
- **KiCad** `kicad/`: IPC to the fork over NNG and protobuf, `kicad_edit` (a whole edit in one tool call) and its router, embedded GUI KiCad per project, plots, Gerber reading, board 3D and STEP meshing (`native/step2glb`), checks (ERC, DRC), BOM, design history steps, installing the fork.
- **Bots** `bots/`: persistent bots, rooms, memory, routines and reminders, mail, webhooks, the uberbot, swarms.
- **Settings** `settings/`: settings as events, the secret vault (AES-GCM, keychain or key file), masks, provider accounts and failover, telemetry, order and fab settings.
- **Other hub lanes**: `session/` (views, composer, search), `commands/` (thread, project, queue, worktree, merge), `files/` (Files tab, git, images, attachments), `term/` (ptys, VT, browser driver), `mcpx/` (MCP servers for agents), `google/` (OAuth, Gmail, Calendar for bots), `voice/` (dictation), `push/` (APNs), `update/` (signed releases), `fab/` and `review/` (fab profiles, design review), `import014/` (import a 0.14 home), `plat/` (Unix and Windows differences).
- **Guard** `native/guard` (`backplane`): starts and watches the hub, restart ladder, rollback, safe mode, answers on the hub's port while it is down. Shares no code with the hub.
- **Voice helper** `native/voice` (`backplane-voice`): a child of the hub, speech to text through OpenAI or local sherpa-onnx, NDJSON on stdout.
- **Desktop client** `native/gpui` (`backplane-gpui`, `src/main.rs`): GPUI 0.2.2 (vendored with one build patch). `hub.rs` is the socket, `ui.rs` the window, `screens/` the hub-described screens, `viewers/` the board, 3D, embedded KiCad (X11 child windows) and embedded Chromium (CEF, feature `cef`).
- **Phone apps** `mobile/ios` (SwiftUI) and `mobile/android` (Compose): speak `data/1` (`docs/PHONE-DATA.md`), pair by link, QR or code.
- **Order server** `order/fabsrv` (FastAPI): quotes, sourcing, payment for the Buy flow; deployed with nginx and systemd.
- **Tools** `tools/browser` (headless Chromium over NDJSON, Bun), `tools/cef` (CEF host).
- **Scripts** `scripts/`: `land.sh` (the only way to main), `dev-build.sh`, `cargo-q` (build slots, Boat offload), `main-build.sh`, `fresh-watch.sh`, `hub-watchdog.sh`, hooks, device rigs.

Data flow: client request (RPC over WebSocket) → handler → `Hub::commit_many` (append, fsync) → model indexes, presenter folds, subscribers told → view deltas or data frames to every client. Agent output streams as live deltas before it becomes an event.

Persistence: `<home>/events.jsonl` (authoritative), checkpoints, `token`, `devices-v1.json`, vault files, per-lane JSON files; git repositories and worktrees of projects; KiCad files committed to main at each turn's end.

External services and dependencies: agent CLIs; the KiCad fork and FreeCAD (headless); Tailscale; GitHub releases (updates, ed25519-signed manifest, vendored `ed25519-compact`); APNs; Google OAuth, Gmail and Calendar; OpenAI speech to text or sherpa-onnx; the order server and its vendor APIs; a telemetry endpoint; Boat sandboxes (builds and always-on hubs); parts lookups.

## Conventions

- **Language and style**: Rust 2021 (hub, guard, voice) and 2024 (client). No `rustfmt.toml`, `clippy.toml`, `deny.toml`, `.editorconfig` or any linter config in the tree. Code is dense with very long lines; several statements or a whole function on one line is normal. Comments explain why, often citing the removed JS source (`src/core/...`) as the specification.
- **Errors**: `anyhow::Result` in the hub. RPC errors go back as `{"t":"reply","id":…,"ok":false,"error":…}`. `.unwrap()` appears 6,747 times in the hub (tests included), `.expect(` 151, `panic!(` 108, `unsafe` 77 (client 196).
- **Logging**: `eprintln!` to stderr (300 in the hub, 140 in the client). No log crate, levels or structured logs in the hub. Health through `/healthz` and the guard's state files. Anonymous usage counts (`settings/telemetry*.rs`), on by default.
- **Tests**: in-crate `#[cfg(test)]` modules (about 3,500 test functions in the hub, 726 in the client), run in CI with nextest in two partitions on Linux, macOS and Windows. e2e drivers in `test/` (shell, Python, TypeScript). Agents in tests are stand-ins only (`BACKPLANE_AGENTS`, `test/tools/claude_standin.ts`). Tests use a temp home, a port from 3799 up and `--no-tailscale`. Many tests are oracles recorded from the old JS hub.
- **Config**: more than 150 distinct `BACKPLANE_*` environment variables read across the hub; hub flags (`--home`, `--port`, `--host`, `--no-tailscale`, `--foreground`, `--default`); settings stored as events (`setting.set`).
- **Process rules**: every child through `proc.rs`; never kill by name or pattern; never write live user data (`~/.backplane*`).
- **Compatibility rules**: persisted events stay decodable forever (add variants, never change one); CBOR dictionary entries are append-only.
- **Product rules**: every feature matches on the GPUI client and both phone apps; the UI is dumb (decisions come from the hub or a tested pure function); no continuous repaint loops; square corners, minimal copy.
- **Git**: one author identity (`build@amemb.com`), fast-forward only through `scripts/land.sh`, which runs the tests first; `main-build.sh` builds every move of `origin/main`.

## Stated intent (the baseline for the missing-features analysis)

From `README.md`, `AGENTS.md` and `docs/`:

1. Eight agent providers in one workspace: Claude, Codex, Grok, OpenCode, Muse Code, Antigravity, Pi, Oh My Pi. Agents start subagents on other providers; threads message each other; an uberbot coordinates bots, projects and threads.
2. Agents edit KiCad through the fork's board and schematic IPC APIs and FreeCAD by script.
3. Viewers: Board, Schematic, 3D, Mech (STEP or GLB); hierarchical schematics; layer controls in KiCad's order and names; cross-probing between views; mechanical-only folders work without a KiCad project.
4. Design history: board and schematic changes are checkpointed; revisit earlier steps, compare versions by sheet and layer, pin review comments; BOM, ERC and DRC from the Review tab.
5. Clients: "Desktop, web and phones. The desktop app, browser, iOS and Android clients connect to the hub." (`AGENTS.md` says there is no web client in this tree.)
6. Install: Linux x64 and arm64 one-liner, macOS Apple silicon (not notarized), Windows x64 experimental; iOS by TestFlight, Android by APK. Signed updates, nightly channel, a guard with a restart ladder, rollback and safe mode.
7. Pairing: link, QR or eight-character code; an invitation expires after ten minutes and works once; devices are individually revocable; machines pair over Tailscale.
8. Catbots: persistent bots with webhooks, routines, a mailbox and a "space" that serves a persistent web app.
9. References with `>`, `#`, `%`, `@`; link cards between threads, bots and projects; inline terminals for TUIs; `/btw` side questions.
10. Secret vault: agents use API keys and tokens by name without seeing their values; output is masked.
11. Voice dictation; push alerts to the phone; git, worktree and PR actions in the thread.
12. Ordering: Buy it opens a thread that checks the design and gets a quote (`docs/order.md`, `order/`).
13. Telemetry: anonymous usage counts, on by default, off in Settings, asked on first run.
14. Skills: KiStack KiCad skills ship with the app.
15. "One hub per machine owns the event log, agent processes, git and file watchers."
16. "The hub is written in [blank] and runs on Bun. Important rules are stated as laws in `LAWS`, with proofs in `PROOF` checked by the compiler." (README; the Bend tree and those files are gone.)
17. Contributing: "Building needs [blank], Bun and Python 3.12+"; `docs/local-testing.md`, `desktop/README.md` (Electron shell), `mobile/README.md`.
18. Guarantees in `AGENTS.md`: events decodable forever; append-only CBOR dictionaries; feature parity across the three clients; the copy guard (five gates, `docs/COPY-GUARD.md`); all process spawning through `proc.rs`; one writer per board or schematic under a named lock; KiCad work committed to main at each turn's end; the hub starts on bad state and quarantines a bad record or checkpoint (`docs/HUB-RESILIENCE.md`).
19. `docs/store-checkpoints.md` describes an SQLite `store.db` with transactional commits; the Rust hub has no SQLite dependency.
20. Phone protocol `data/1` as specified in `docs/PHONE-DATA.md` (72 KB).

## Excluded from review

| What | Why |
|---|---|
| `native/gpui/vendor/gpui`, `native/gpui/vendor/xattr`, `native/host/vendor/ed25519-compact`, `native/voice/sherpa/c-api.h`, `native/voice/vad` | vendored third-party code (the patch notes are read in the cross-cutting pass) |
| `deploy/kistack/**` | skills pinned from the KiStack repository by `deploy/kistack.lock` |
| `Cargo.lock` (4), `tools/browser/bun.lock`, `deploy/*.lock*` | lockfiles (read for dependency state only) |
| `mobile/android/.../Msg.kt`, `mobile/ios/Shared/TrCatalog.swift`, `native/host/src/settings/zones_table.rs`, `native/host/src/kicad/strokefont_data.rs`, `native/gpui/vendor/gpui/windows-shaders/shaders_bytes.rs` | generated code or data tables |
| `docs/media/**`, `docs/viewer-shots/**`, `assets/fonts/**`, `assets/icon/**`, every `png`, `wav`, `glb`, `step`, `ttf` | binary and media assets |
| `test/files/**`, `test/**/fixtures/**`, `native/host/src/present/fixtures/*.json`, `test/kipc_bench` data, `docs/*.json` reports | fixtures, oracles and recorded data |
| `i18n/*.msg`, `assets/web/style.css` | catalogs and the colour source (checked by scripts, not read line by line) |
| `deploy/licenses/**`, `native/gpui/LICENSES`, `LICENSE` | licence texts |
| `docs/**/*.md` | read for stated intent and drift, not reviewed line by line |

## Scope decision: the tree is too large to review all of it in depth

First-party code is about 18 MB (about 4.5 million tokens to read once): hub 11.3 MB, client 3.1 MB, phone apps 2.1 MB without the generated catalogs, scripts 0.7 MB, tests' drivers about 3 MB. At 80 to 250 KB a unit that is about 80 units.

- **Tier 1, reviewed in full (units 01 to 41)**: the hub's store, wire, auth, process safety, net, state, session, commands, phone data, agents, settings and secrets, files, terminals, bots, the KiCad write paths, update and guard, MCP, Google, fab and order, the order server, the client's connection and pairing code, the landing, build and release scripts, CI, both phone apps' connection code, voice, the browser tool.
- **Tier 2, reviewed in full after tier 1 (units 42 to 56)**: KiCad checks, BOM, plots, Gerber and 3D, the sidebar and plan folds, the presenter, agent prompts and skills, the client's window, input and viewers, the shipped skills.
- **Tier 3, sampled (units 57 to 61)**: the client's 77 screen files (the largest and most-changed are read), the phone apps' view code, the test harness.
- **Skipped**: the hub's in-crate test files as code (each unit's reviewer judges the tests of its unit instead), device and screenshot rig scripts (`scripts/desktop-*`, `scripts/ios-*`, `scripts/android-*`, `scripts/mobile/*`), most of `test/**` drivers.

`coverage.md` has the file list and status of each unit.

## Cross-cutting concerns to check across units

1. **Authentication on every route**: the pairing token or device credential is "checked first" for HTTP handlers (`api.rs`); which routes are exempt (`/pair`, `/healthz`, `/auth/enroll`, webhooks, `/voice/in`, the loopback MCP server, bot spaces) and whether each exemption is safe.
2. **Loopback trust**: the agent MCP server and other loopback ports; what another local user or a web page in the embedded browser can reach.
3. **The event contract**: every writer of events against every fold (`model.rs`, `state/`, `sidebar.rs`, `present/`, `data/`); decodability of old events; checkpoint stamps covering every fold input.
4. **Commit failure**: what each caller does when `commit_many` fails (ack before commit, partial state, effects run on a failed commit).
5. **The copy guard**: every spawn through `proc.rs` with the right `Kind`; turn starts gated; outbound HTTP made without a child process (in-process clients) and whether the guard covers it.
6. **Secrets**: the vault, masks applied to every output path (tool results, logs, terminals, events, telemetry, issue reports, diagnostics), tokens in URLs (`?token=`), credentials at rest on the client and phones.
7. **Path handling**: file serving, attachments, images, git operations, imports and archive extraction (zip, tar) against traversal and symlinks.
8. **Shell and argument injection**: embedded scripts in `files/scripts/`, `sh -c` use, branch, file and project names reaching a shell.
9. **Locks and concurrency**: `Mutex<Model>` held across awaits, actor mailboxes, the design-file locks, broadcast lag handling, unbounded queues and buffers.
10. **Wire compatibility**: `wire_dict.rs` order shared by hub and client (`native/gpui/src/wire_dict.rs`), `data/1` shapes shared by hub, iOS and Android.
11. **Parity**: features present on one client and missing on another.
12. **Updates**: signature checks, downgrade and channel rules, the guard's rollback, the install scripts.
13. **Platform differences**: Unix-only code paths reached on Windows, and macOS versus Linux (systemd scopes, X11 embedding).
14. **Docs against behaviour**: the README and docs that still describe the removed tree.
15. **Error propagation style**: `.unwrap()` on input that comes from outside the process (network, files, agent output).

## Models used for review

Added on the user's instruction during phase 2. Three bands, recorded per unit in `coverage.md`:

- Fable 5.2 (`claude-fable-5-2`, the most capable): the high-risk units and phases 3 to 5.
- Opus 5.5 (`claude-opus-5-5`): medium-risk units, where a failure is a wrong picture, a wrong check result or a viewer crash.
- Sonnet 5.5 (`claude-sonnet-5-5`): low-risk units (tooling, forms, prompts, skills, test harness, samples).

A Confirmed High or Critical finding from an Opus or Sonnet unit is treated as Possible until it is re-traced in phase 3.
