# Coverage

Commit `9c6c5448a514ebcddca5a776933793e25179a796` on `main`. Status: pending, running, done, sampled, skipped (with reason).

Models: `fable-5.2` (claude-fable-5-2, the most capable) reviews the high-risk units (a missed bug means data loss, an authentication bypass, a leaked secret or a wrong write to design files) and does phases 3 to 5. `opus-5.5` (claude-opus-5-5) reviews the medium-risk units (real logic, concurrency or unsafe code, where a failure is a wrong picture, a wrong check result or a viewer crash). `sonnet-5.5` (claude-sonnet-5-5) reviews the low-risk units (tooling scripts, presenter forms, prompts, shipped skills, the test harness, the tier 3 samples). A Confirmed finding of High or Critical severity from an `opus-5.5` or `sonnet-5.5` unit counts as Possible until it is re-traced in phase 3. Mechanical work (inventory, line counts, pattern scans, running tests) was done by shell commands, not by a model.

`H/` means `native/host/src/`, `G/` means `native/gpui/src/`, `A/` means `mobile/android/app/src/main/java/dev/backplane/mobile/`, `I/` means `mobile/ios/Backplane/`. Sizes are source bytes in KB.

## Tier 1: reviewed in full

| # | Unit | KB | Files | Model | Status |
|---|---|---:|---|---|---|
| 01 | hub-store-core | 195 | H/store.rs, hub.rs, model.rs, api.rs, ckpt.rs, ckptrun.rs, fold_inputs.rs, cbor.rs, delta.rs, stamp.rs, startup.rs; native/host/build.rs | fable-5.2 | done |
| 02 | hub-edge-wire-auth | 127 | H/main.rs, wire/mod.rs, wire/tests.rs, wire_dict.rs, auth.rs, presenter.rs, window.rs, bin/*.rs; native/host/Cargo.toml | fable-5.2 | done |
| 03 | hub-process-safety | 160 | H/proc.rs, copy.rs, guard.rs, memscope.rs, plat/*.rs | fable-5.2 | done |
| 04 | net-pairing-devices | 223 | H/net/mod.rs, pairing.rs, pairing/tests.rs, pairflow.rs, pairflow/tests.rs, routes.rs, devices.rs, devicespage.rs, qr.rs, sign.rs, owner.rs, origin.rs, start.rs, listener.rs, tailnet.rs, lan.rs | fable-5.2 | done |
| 05 | net-far | 222 | H/net/far.rs, farfiles.rs, farsend.rs, farstore.rs, farsync.rs, farsync/*.rs, farview.rs, feed.rs, feed/tests.rs, outbox.rs, outbox/tests.rs, rpc.rs, rpc/tests.rs | fable-5.2 | done |
| 06 | net-peers-registry | 132 | H/net/peerhttp.rs, peering.rs, peering/tests.rs, peerrpc.rs, peers.rs, mesh.rs, machines.rs, machinespage.rs, botnet.rs, copy_outs.rs, plan.rs, registry.rs | fable-5.2 | done |
| 07 | net-alwayson-homemove | 251 | H/net/alwayson/*.rs, H/net/homemove.rs | fable-5.2 | done |
| 08 | state-fold-policy | 260 | H/state/*.rs | fable-5.2 | done |
| 09 | session-views | 272 | H/session/*.rs | fable-5.2 | done |
| 10 | commands | 286 | H/commands/*.rs | fable-5.2 | done |
| 11 | phone-data-push | 299 | H/data/*.rs, H/push/*.rs | fable-5.2 | done |
| 12 | agents-core | 229 | H/agents/mod.rs, proc.rs, launch.rs, rules.rs | fable-5.2 | done |
| 13 | agents-turn-support | 256 | H/agents/lines.rs, inputs.rs, halt.rs, halt_view.rs, hist.rs, host.rs, wake.rs, work.rs, tasks.rs, bg.rs, settle.rs, cmds.rs, codex.rs, devturns.rs, far.rs, recall.rs, review.rs, laws.rs | fable-5.2 | done |
| 14 | agents-mcp-tools-locks | 182 | H/agents/mcp.rs, mcp_stdio.rs, tools.rs, account_tools.rs, design.rs, locks.rs, lockrec.rs, kicadsel.rs, world.rs, person.rs | fable-5.2 | done |
| 15 | agents-providers | 299 | H/agents/providers/* | fable-5.2 | done |
| 16 | settings-vault-telemetry | 290 | H/settings/vault.rs, secret.rs, mask.rs, keys.rs, webhook.rs, privacy.rs, copy_gate.rs, set.rs, mod.rs, misc.rs, binding.rs, alerts.rs, mentions.rs, zone.rs, telemetry.rs, telemetry_send.rs, telemetry_send_tests.rs, telemetry_usage.rs, diagnostics.rs, diagnostics_http.rs, issue_report.rs | fable-5.2 | done |
| 17 | settings-accounts-io | 183 | H/settings/accounts_io.rs | fable-5.2 | done |
| 18 | settings-accounts-models | 185 | H/settings/accounts.rs, accounts_hub.rs, accounts_people.rs, accounts_phone.rs, accounts_sync.rs, accounts_there.rs, claude_usage.rs, models.rs, models_probe.rs | fable-5.2 | done |
| 19 | settings-failover-driver | 353 | H/settings/driver.rs, fallback.rs, onboard.rs, tools_install.rs, skill_install.rs, improvements.rs, proposal.rs, e2e.rs, failover_e2e.rs, restart_e2e.rs | fable-5.2 | done |
| 20 | files-git | 270 | H/files/** (Rust and the embedded scripts in files/scripts/) | fable-5.2 | done |
| 21 | term | 266 | H/term/** | fable-5.2 | done |
| 22 | bots-core | 237 | H/bots/api.rs, rpc.rs, tools.rs, tools.json, gate.rs, hooks.rs, world.rs, plane.rs, post.rs, reply.rs, front.rs, life.rs, mod.rs | fable-5.2 | done |
| 23 | bots-sched-swarm | 236 | H/bots/clock.rs, routines.rs, sched_text.rs, swarm.rs, ring.rs, uber.rs, mesh.rs, far.rs, relocate.rs, ledger.rs, ledger.json | fable-5.2 | done |
| 24 | bots-mail-recall | 263 | H/bots/mail/*.rs, H/bots/recall/*.rs | fable-5.2 | done |
| 25 | kicad-embed | 235 | H/kicad/embed.rs | fable-5.2 | done |
| 26 | kicad-edit-route | 259 | H/kicad/kedit.rs, kroute.rs, edchecks.rs | fable-5.2 | done |
| 27 | kicad-ipc | 247 | H/kicad/ipc.rs, kipc.rs, kserve.rs, nng.rs, proto.rs, coedit.rs, savegate.rs, reload.rs, open.rs, bj.rs, hostlibs.rs, libtables.rs | fable-5.2 | done |
| 28 | kicad-install-watch | 256 | H/kicad/install.rs, watch.rs, canonical.rs, project.rs, boards.rs, mod.rs, cad.rs, progress.rs, mgmt.rs, release.rs, kexport.rs | fable-5.2 | done |
| 29 | update-and-guard | 205 | H/update/**, native/guard/** | fable-5.2 | done |
| 30 | mcpx-md | 279 | H/mcpx/*.rs, H/md/*.rs | fable-5.2 | done |
| 31 | google-import014 | 237 | H/google/*.rs, H/import014/*.rs | fable-5.2 | done |
| 32 | fab-order-hub | 286 | H/fab/**, H/settings/fab.rs, order.rs, order_view.rs | fable-5.2 | done |
| 33 | order-server | 165 | order/** | fable-5.2 | done |
| 34 | client-core-net | 226 | G/main.rs, hub.rs, hubs.rs, pairing.rs, launch.rs, local.rs, install.rs, delta.rs, wire.rs, wire_dict.rs, plat.rs, tmp.rs, bin/*; native/gpui/Cargo.toml | fable-5.2 | done |
| 35 | scripts-land-build | 190 | scripts/land.sh, dev-build.sh, dev-cut.sh, dev-reload.sh, cargo-q, main-build.sh, main.hook, pre-commit.hook, pre-push.hook, check-author.sh, install-hooks.sh, lib-bptmp.sh, lib-buildslot.sh, lib-boat.sh, bp-build.sh, bp-fetch.sh, bp-lock.sh, bp-slot.sh, boat-pool.sh, boat-run.py, boat-base.sh, boat-ssh.sh, boat-prune.py, boat-artifacts.py, worktree-gc.sh, fresh.sh, fresh-watch.sh, hub-watchdog.sh, smoke-start.sh, switch.sh, side-by-side.sh, run-here.sh, sccache-tmp, systemd/* | sonnet-5.5 | done |
| 36 | release-install-ci | 210 | .github/workflows/*.yml; deploy/install.sh.in, compat/*, windows/*, aur/*, backplane-launch.sh, backplane.desktop, bun-version, kistack.lock, kicad-library.lock.json; scripts/nightly-update-gate.sh, bare-gate.sh, bare-gate-nightly.sh, licenses.sh, rust-licenses.sh, migrate-from-bend.sh, build-*.sh, kistack.py, check-linux-archive.sh, ci-hub-windows-filter.sh, runner-nightly-only.sh, mac-build.sh, mac-hub-build.sh, cef-runtime.py, cef-runtime-files.sh, cef-mac-helpers.sh, windows-shaders.sh, windows-check.sh, bun-pin.sh, fetch-fonts.sh | fable-5.2 | done |
| 37 | android-net-core | 205 | A/DataClient.kt, Hub.kt, HubList.kt, HubRescue.kt, HubScreen.kt, Pair.kt, Scanner.kt, Outbox.kt, LiveService.kt, TermClient.kt, TermStream.kt, AppSettings.kt, DataModels.kt, ScreenCache.kt, DataNotify.kt, DataMigrate.kt, GoogleLoop.kt, VoiceRecorder.kt, App.kt, DataActivity.kt, DataRing.kt, DataDevices.kt, DataMachines.kt; mobile/android/app/src/main/AndroidManifest.xml, app/build.gradle.kts | fable-5.2 | done |
| 38 | ios-net-core | 215 | I/DataClient.swift, Hub.swift, HubList.swift, HubRescue.swift, HubView.swift, Pair.swift, PairView.swift, PairingStore.swift, Outbox.swift, DataPush.swift, AppSettings.swift, DataModels.swift, TermStore.swift, TermStream.swift, Migration.swift, GoogleLoop.swift, GoogleLoopLogic.swift, VoiceRecorder.swift, WireJSON.swift, Stall.swift, AppIdentity.swift, DataApp.swift, DataNotify.swift, DataRing.swift, DataDevices.swift, DataMachines.swift, DataImageSave.swift; mobile/ios/Backplane.entitlements, Info.plist.in, build.sh, Backplane/PrivacyInfo.xcprivacy | fable-5.2 | done |
| 39 | hub-voice | 288 | H/voice/** | opus-5.5 | done |
| 40 | voice-helper | 200 | native/voice/src/*.rs, Cargo.toml, README.md, examples/* | fable-5.2 | done |
| 41 | tools-browser-cef | 130 | tools/browser/src/*.ts, build.ts, package.json, README.md; tools/cef/host.cc, CMakeLists.txt, README.md; tools/platform/windows-path.ts | opus-5.5 | done |

## Tier 2: reviewed in full after tier 1

| # | Unit | KB | Files | Model | Status |
|---|---|---:|---|---|---|
| 42 | kicad-checks-history | 265 | H/kicad/checkrun.rs, checks.rs, findings.rs, review.rs, comment.rs, histsteps.rs, parthist.rs, lifepanel.rs, xprobe.rs | opus-5.5 | done |
| 43 | review-bom-parts | 242 | H/review/*.rs, H/kicad/bom.rs, bomedit.rs, bompanel.rs, parts.rs | fable-5.2 | done |
| 44 | kicad-plot-gerber | 284 | H/kicad/plot.rs, plotgen.rs, plotread.rs, png.rs, render.rs, schematic.rs, board.rs, sexpr.rs, strokefont.rs, fl.rs, gerber/*.rs | opus-5.5 | done |
| 45 | kicad-3d-step | 346 | H/kicad/pcb3d/*.rs, glb.rs, mesh.rs, solid.rs, solidread.rs, stepglb.rs; native/step2glb/* | opus-5.5 | done |
| 46 | sidebar-signal-plan | 150 | H/sidebar.rs, signal.rs, shelf.rs, mirror.rs, plan/*.rs | opus-5.5 | done |
| 47 | present-screen-thread | 256 | H/present/screen.rs, thread.rs, mod.rs, merged.rs | opus-5.5 | done |
| 48 | present-settings-onboard | 261 | H/present/settings.rs, onboard.rs, onboard_alwayson.rs, signin_there.rs | sonnet-5.5 | done |
| 49 | present-viewer-panels | 233 | H/present/viewer.rs, panels.rs, picker.rs, keys.rs, window.rs | sonnet-5.5 | done |
| 50 | present-sections-misc | 221 | H/present/sections.rs, sections_tests.rs, tokstats.rs, botforms.rs, swarmform.rs, native_sidebar.rs, filelink.rs, filelink_tests.rs, linkopen.rs, badge.rs, ready.rs, tz.rs, refusal.rs, audit_files.rs | sonnet-5.5 | done |
| 51 | agents-prompts-skills | 147 | H/agents/skills.rs, bundled.rs, refs.rs, mailroom.rs, prompt.rs, prompt_text.rs, textgen.rs, iconpick.rs, groupname.rs | sonnet-5.5 | done |
| 52 | client-ui-input | 264 | G/ui.rs, keys.rs, input.rs, script.rs | opus-5.5 | done |
| 53 | client-viewers-kicad | 240 | G/viewers/kicad_pane.rs, native_child.rs | opus-5.5 | done |
| 54 | client-viewers-3d | 200 | G/viewers/solid.rs, solid_gpu.rs, solid.wgsl, raster.rs, boards.rs, layers.rs, demo.rs | sonnet-5.5 | done |
| 55 | client-browser-cef-misc | 291 | G/viewers/cef.rs, cef_native.rs, cef_page.rs, cef_mac.rs, chrome.rs, browser.rs, downloads.rs, mod.rs; G/streamer.rs, i18n.rs, i18n_scan.rs, art.rs, perf.rs, part.rs, swatch.rs, theme.rs, scale.rs | opus-5.5 | done |
| 56 | shipped-skills | 150 | deploy/skills/sendcutsend/**, skills/backplane-e2e/** | sonnet-5.5 | done |

## Tier 3: sampled

| # | Unit | KB read / total | Files read | Model | Status |
|---|---|---:|---|---|---|
| 57 | client-screens-a | 232 / 1,744 | G/screens/viewer.rs, timeline.rs, composer.rs, term.rs | sonnet-5.5 | sampled (done) |
| 58 | client-screens-b | 238 / 1,744 | G/screens/projects.rs, settings.rs, accounts.rs, pair.rs, hubpick.rs, onboard.rs | sonnet-5.5 | sampled (done) |
| 59 | android-views | 215 / 800 | A/DataViews.kt, DataViewer.kt, Vt.kt, DataMarkdown.kt, DataLinks.kt, DataLinkGrammar.kt | sonnet-5.5 | sampled (done) |
| 60 | ios-views | 225 / 850 | I/DataViews.swift, DataViewer.swift, DataVoice.swift, Vt.swift, DataLinks.swift, DataLinkGrammar.swift | sonnet-5.5 | sampled (done) |
| 61 | test-harness | 200 / 3,000 | test/tools/*.ts, test/guard/*, test/e2e/lib/*, test/update/* (drivers), test/hub/*.sh | sonnet-5.5 | sampled (done) |

## Skipped

| What | KB | Reason |
|---|---:|---|
| Hub in-crate test files as code: H/agents/tests.rs, agents/tests/*, agents/core_e2e.rs, H/bots/tests.rs, bots/tests/* | 600 | Test code. Each unit's reviewer judges the tests of its own unit for gaps instead of line-reviewing them. |
| G/screens: the other 67 files | 1,270 | View code described by the hub ("the UI is dumb"); the ten largest and most-changed files are sampled in units 57 and 58. |
| Phone view code not listed in 59 and 60 | 1,200 | Same reason; connection, pairing and storage code is in units 37 and 38. |
| scripts/desktop-*, ios-*, android-*, mobile/*, lat/*, android-harness/*, i18n-*.py, check-*-i18n.py, misc one-off rigs | 400 | Developer rigs and screenshot drivers that run on the maintainers' machines; a pattern scan for dangerous commands is done in the cross-cutting pass. |
| test/** drivers other than unit 61 | 2,800 | e2e and visual drivers; sampled in unit 61. |
| Everything in the "Excluded from review" table of 00-map.md | | Vendored, generated, lockfiles, media, fixtures, catalogs, licence texts. |

## Phases 3 to 5 (all on fable-5.2)

| Step | Output | Status |
|---|---|---|
| Re-trace of High findings from Opus and Sonnet units, and spot checks of Fable findings | `_verify.md` | done: 20 rows re-traced (12 kept High, 8 lowered to Medium, none refuted), 6 spot checks |
| Tests, catalog checks, secrets scan, CI and dependency notes | `_notes-phase3.md` | done |
| Cross-unit questions (489, in seven area files under `_questions/`) | `91-questions-*.md` | done: 62 no issue, 281 confirm existing findings, 72 led to new findings (74 rows), 12 open |
| Cross-cutting pass | `90-cross-cutting.md` | done |
| Missing features | `95-missing-features.md` | done |
| Findings register (every finding in full, generated from the unit files) | `findings-all.md` | done |
| Final report | `REVIEW.md` | done |

## Files in this folder

| File | What it is |
|---|---|
| `REVIEW.md` | The final report |
| `findings-all.md` | Every finding of every severity in full text, duplicates merged |
| `00-map.md` | The map of the codebase, stated intent, scope decision |
| `coverage.md` | This table |
| `NN-<unit>.md` (01 to 61) | One file per review unit: summary, findings, improvements, feature gaps, missing tests, questions, style notes |
| `90-cross-cutting.md` | The cross-cutting pass |
| `91-questions-*.md` | Answers to the reviewers' cross-unit questions, with new findings and interface mismatches |
| `95-missing-features.md` | Missing features: promised, partial, inferred |
| `_verify.md`, `_notes-phase3.md` | The re-trace log and the tool-run notes that fed `90-cross-cutting.md` |
| `_reviewer-brief.md`, `_unit-prompts.md`, `_questions-brief.md`, `_questions/` | What each reviewer was asked to do |
