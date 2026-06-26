# No-Ads & No-Sponsors Cleanup Design

**Date**: 2026-06-26
**Branch target**: `cleanup/no-ads` (from `main` at origin/main, which includes 10 commits newer than the previous spec baseline, notably 5 new sponsor additions and 4 paste-fix commits)
**Status**: Draft — pending user review
**Scope**: Remove all in-repo advertising, sponsor promotion, donation prompts, and external promotional links. Relay / mobile-relay / provider-preset code is preserved by explicit user decision.

## 1. Background

The `main` branch ships with three layers of promotional content (some newly added since the previous cleanup attempt):

- A **remote ad list** (`BigPizzaV3/Ad-List`) fetched at runtime by the Rust helper and rendered in both the Tauri manager and the Codex-side inject script.
- **Sponsor logos and write-ups** in `README.md` and `README_EN.md` (now 14 sponsor rows including 2 new: Sui Xiang AI Gateway, Yiyun Technology), with a JOJO Code promo banner.
- **Donation QR codes** (Alipay / WeChat) embedded in the README, the Tauri manager overview panel, and the inject-script support tab.
- A "Recommended content" tab in both the manager UI and the inject-script modal.
- A "Friendly links" section linking to LINUX DO at the bottom of the README.
- **Sponsor logo image files** under `docs/images/sponsor-*` and `assets/images/sponsor-*` (now 21 files including the 2 new ones: `sponsor-sui-xiang-ai-gateway.jpg`, `sponsor-yiyun-tech.jpg`).

## 2. In Scope vs Out of Scope

### 2.1 In scope (will be removed)

**Rust core** (`codex-plus-core`):
- `src/ads.rs` (delete file)
- `src/lib.rs` — remove `pub mod ads;`
- `src/routes.rs` — remove `async fn ads` trait method, `/ads` route, and impl
- `src/assets.rs` — remove `SPONSOR_ALIPAY`/`SPONSOR_WECHAT` const, `sponsor_image_data_uris()`, and the `__CODEX_PLUS_SPONSOR_IMAGES__` slot in `injection_script_with_settings`
- `tests/ads.rs` (delete file)
- `tests/bridge_routes.rs` — remove `async fn ads` mock impl and the `("/ads", …)` test entry
- `tests/cdp_bridge.rs` — remove `__CODEX_PLUS_SPONSOR_IMAGES__` assertion, `injection_script_fetches_ads_without_bridge` test, and rename the helper-URL test

**Tauri manager** (`codex-plus-manager`):
- `src-tauri/src/commands.rs` — remove `AdsPayload`, `load_ads`, `ads_payload()`, and the `ads_payload_keeps_version_and_ad_items` test
- `src-tauri/src/lib.rs` — remove `commands::load_ads` from invoke handler list
- `src/App.tsx` — remove `AdItem`/`AdsResult` types, `isExpiredAd`, `ads` state, `refreshAds` action, `refreshAds` field on `Actions`, `"recommendations"` route + sidebar entry, the `route === "recommendations"` JSX branch, JOJO Code promo `<Panel className="jojocode-overview">`, `RecommendationsScreen`, `AdGrid`, i18n `recommendations` label, and the "推荐内容" reference in the `## 增强功能` description (line "Timeline, 推荐内容和用户脚本等增强...")
- `src/styles.css` — remove `.jojocode-overview*`, `.jojocode-model-tags*`, `.recommend-hero*`, `.ad-grid`, `.ad-card`, `.ad-card:hover`, `.ad-card strong`, `.ad-card p`, `.ad-tags`, `.ad-tags span`, `.ad-link`

**Launcher** (`codex-plus-launcher`):
- `src/main.rs` — remove `async fn ads` impl

**Inject script** (`assets/inject/renderer-inject.js`):
- `codexPlusAdsUrl`, `codexPlusAds`, `codexPlusAdsLoaded`, `isCodexPlusAdExpired`, `normalizeCodexPlusAds`, `renderCodexPlusAdGroup`, `renderCodexPlusAds`, `cacheBustCodexPlusAdUrl`, `directFetchCodexPlusAds`, `fetchCodexPlusAds` (delete all)
- Modal `sponsor` and `support` tab buttons, both panels, and the orphan `[data-codex-plus-active-tab="support"]` CSS rule
- All `codex-plus-sponsor-*` and `codex-plus-ad-*` CSS rules

**Image assets**:
- `assets/images/sponsor-alipay.jpg`, `sponsor-wechat.jpg`, `rawchat-sponsor.jpg`
- `docs/images/` — 18 sponsor-*.{svg,png,jpg} files including the 2 new ones (`sponsor-sui-xiang-ai-gateway.jpg`, `sponsor-yiyun-tech.jpg`); all files matching `docs/images/sponsor-*`

**README**:
- `README.md` — entire `## 赞助商` section (now includes 14 rows: 12 original + Sui Xiang + Yiyun), donation paragraph + Alipay/WeChat QR images in `## 交流与支持`, `## 推荐内容` section, LINUX DO entry under `## 友情链接`, and the `推荐内容` mention in `## 增强功能` description
- `README_EN.md` — equivalent `## Sponsors`, `## Recommendations`, donation block, `## Friendly Links` sections, and the `recommendations` mention in the Enhancements description

### 2.2 Explicitly preserved

- **Relay core** (`crates/codex-plus-core/src/relay_config.rs`, `relay_switch.rs`, `relay_rotation.rs`, `protocol_proxy.rs` and their `tests/relay_*` counterparts)
- **Mobile relay app** (`apps/codex-plus-mobile-relay/`)
- **Provider presets** in `apps/codex-plus-manager/src/presets.ts` — all 10 aggregator entries (jojocode, siliconflow, openrouter, aihubmix, apikeyfun, pateway, therouter, novita, shengsuanyun, ccsub) and other categories
- **Relay-related UI** in `App.tsx` (`relay`, `mobileControl` routes, `ProviderPresetSelector`)
- **Brand name** `CodexPlusPlus`
- **paste-fix** features (newly merged from `feat/paste-fix-integration`) — these are functional code unrelated to ads/sponsors
- **`CONTRIBUTING.md`** and **`.github/ISSUE_TEMPLATE/provider_config.yml`**

## 3. Approach

**彻底删除 (full removal)** — same as before. Reuse previously approved decisions:
- 5 themed commits on one branch
- No backup files, no no-op stubs
- Brand name unchanged

## 4. Implementation Plan (5 commits)

1. `chore(core): remove ads module and sponsor image asset plumbing` (incl. bridge_routes test mock cleanup that should have been in Task 1 last time)
2. `chore(manager): remove recommendations screen, JOJO Code promo panel, ad styles`
3. `chore(inject): remove sponsor tab and ad-fetch script` (incl. cdp_bridge test assertion cleanup + orphan support CSS)
4. `chore(assets): remove sponsor logo images` (now 21 files instead of 19)
5. `docs: remove sponsors, ads, donation QR codes and friend links from READMEs` (now 14-row table + 2 new sponsors)

## 5. Verification

`cargo fmt --check`, `cargo test --workspace`, `cargo build -p codex-plus-launcher --release`, `cd apps/codex-plus-manager && npm run check && npm run build` must all pass.

Post-cleanup `git grep` for sponsor/donation/ads keywords should match only the preserved `jojocode`/`apikeyfun` aggregator entries in `presets.ts` and unrelated test fixtures (`max2.jojocode.com` in `relay_config.rs`).

## 6. Risks

- The pre-existing `settings::tests::settings_store_save_load_roundtrip_preserves_aggregate_relay_settings` test failure (also fails on `main`) is **not** caused by this branch.
- `bridge_routes.rs` test mock cleanup and `cdp_bridge.rs` test assertion cleanup are now explicitly in scope (Task 1 and Task 3 respectively) to avoid build breaks.

## 7. Out of Scope

- Re-packaging the macOS DMG (separate task after merge)
- Renaming the project brand
- Re-introducing any sponsorship surface
- `presets.ts` aggregator list

## 8. Rollback

5 linear commits on a single branch. `git reset --hard main` discards all of them. The spec/plan documents on `main` are force-added and survive `git reset`.
