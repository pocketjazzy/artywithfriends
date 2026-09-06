# Arty With Friends — Roadmap

Fork of [WARDOGS Artillery Calculator](https://github.com/apollyon-sys/wardogs-calculator) by Apollyon (MIT, with permission). This file tracks what the fork adds on top of upstream. Everything upstream already does is out of scope here; see the upstream docs for that.

Revised 2026-09-06 after the fork. The original from-scratch plan lives in the `wd-ranger` repo and is superseded.

---

## What the fork already gave us (done)

Everything Phases 1–4 of the old plan were going to build, and more:

- Mortar and SPH-2 range, MIL and azimuth from full in-game firing tables (they match every reading we took ourselves).
- In-range / out-of-range status, LOW/HIGH SPH-2 solutions, Terrain3D elevation correction (opt-in).
- Interactive tiled maps (Bakurani, Ozeti, custom), coordinate search, ruler, drawing, markers.
- Saved targets with names, export/import.
- Mobile-first touch UI at `/mobile/`, twelve languages.
- Static hosting on GitHub Pages, live at **https://artywithfriends.com**.

Fork-only changes, done: no analytics, no ads, no donations; attribution and MIT kept; branding and domain.

---

## Phase A — Baseline (DONE 2026-09-06)

Fork, `site` branch, attribution, tracker removed, domain, Pages deploy, HTTPS. `main` tracks upstream unchanged so pull requests stay clean.

---

## Phase B — Group / multiplayer

### The situation upstream (read this first)

- Upstream issue [#5](https://github.com/apollyon-sys/wardogs-calculator/issues/5) asks for live sharing over websockets. **Apollyon has endorsed it**: unguessable room link, anyone with the link edits, Cloudflare Worker + Durable Object using the WebSocket Hibernation API, batch updates, no accounts or permissions in v1.
- Contributor **Crecket** already has it working on his fork (`Crecket/wardogs-calculator`, branch `feat/collab-rooms`, live at wardogs-map.olm.pet) and is extracting it into upstream PRs. His design doc is `docs/collaboration.md` on that branch. It covers most of our spec already:

| Our spec | Crecket's rooms |
|---|---|
| Random unguessable code, no fingerprinting, relay-enforced uniqueness | Yes — 12 chars, 31-symbol alphabet minus i/l/o/0/1, minted server-side |
| Survives reload | Yes — code rides in the share link (`#room=<code>`) |
| Nicknames | Yes — typed on join, shown in a roster |
| Unique colour per member | Yes — colour per peer, used for live cursors |
| Shared target list, each viewer computes their own solution | Yes — saved targets, artillery/target positions, drawings and markers are shared; zoom/pan/layers stay local |
| Relay on Cloudflare, in-memory, low cost | Yes — Worker + Durable Object, hibernation, SQLite-backed, per-fork relay URL injected at build time |
| Limits | 16 peers + 8 read-only viewers, room lives 2 weeks after last change |
| Optional 4-digit passcode (dropped for now) | No — the code is the only credential |
| **View-only vs collaborate mode** | **Partly** — a read-only "viewer" flag exists for the OBS overlay, not as an owner-set mode |
| **Creator's name on each target** | **No** |
| **Default rank+animal nicknames** | **No** — blank until typed |
| Cap 35, 3-hour lifetime | Different numbers (16 peers, 2 weeks) — trivially configurable |

**Decision (recommended): do not build a third room implementation.** Build on Crecket's work and contribute the gaps. That keeps one community tool, which is what we told Apollyon, and it gets us a working group feature weeks sooner.

### B1 — Evaluate (next)

1. Deploy Crecket's `sync/` Worker to our own Cloudflare account (his README: `cd sync && npm install && npx wrangler login && npm run deploy`; drop his custom-domain route, use our `workers.dev` hostname or `sync.artywithfriends.com`).
2. Build his branch locally against that relay (`COLLAB_URL=wss://... npm run build`) and test a room with two browsers, then with friends.
3. Decide: (a) adopt his whole branch into our fork now for testing with friends, accepting we're ahead of upstream and will rebase later; or (b) wait for his PRs to land upstream and take only those. (a) is the faster route to a test session; (b) is cleaner. Likely (a) for a `collab` test branch and (b) for what ships on `site`.

### B2 — Relay on our Cloudflare

Whatever the branch decision, the relay is ours to run: Worker + Durable Object from `sync/`, configured with our limits (35 peers, 3 h room lifetime). Free plan is viable; $5/mo Workers Paid if usage grows. `npx wrangler login` on this PC is the only credential needed — never API keys in chat.

### B3 — Our additions (as PRs to Crecket's branch or upstream, whichever is live)

In order of value (decided 2026-09-06):

1. **Creator's nickname on each shared target**, in the creator's colour.
2. **View-only / collaborate mode** as a creator-set switch, building on the existing viewer flag.
3. **Default nicknames**: military rank + animal from a large list, editable, remembered on the device.
4. Configurable peer cap and room lifetime (already server-side constants).

Dropped for now: the optional 4-digit passcode. The room code is unguessable on its own; revisit only if uninvited joins turn out to be a real problem.

Coordinate on issue #5 before starting so Crecket and Apollyon know what's coming; ask which pieces they'd rather do themselves.

### B4 — Upstream

Once Apollyon merges the base room sync, the additions above go up as small separate PRs. Our `site` branch then carries only branding + our relay URL.

---

## Phase C — Voice input (our distinctive feature; nobody upstream is doing this)

Goal: hands stay on the keyboard; the player speaks coordinates to a phone or tablet and hears the solution.

1. **Web Speech API on the existing mobile UI** (`/mobile/`): a hold-to-talk button, spoken-number parser ("one oh seven point seven eight" → 107.78, "point"/"dot", "oh"/"zero"), read-back of the parsed coordinates before speaking the range and MIL. Requires HTTPS (done).
2. **Voice for the group feature**: "target north bunker" looks up a named shared target.
3. **PWA polish**: manifest + service worker so the mobile UI installs to the home screen and works offline (tiles permitting). Validate the voice UX here before spending on app stores.

Upstream-worthy once it works: it's a self-contained module. Offer it as a PR.

---

## Phase D — Native apps

- **Android**: wrap the built site with Capacitor; Android Studio on Windows; Play developer account $25 one-time. Use a native speech plugin only if the WebView's speech API proves unreliable.
- **iOS**: same Capacitor project; needs Xcode on macOS (cloud Mac or CI runner) and the $99/yr Apple programme. The PWA from Phase C already serves iPhone/iPad users in Safari before any store app exists.

---

## Housekeeping

- **Map tiles to R2.** The Pages artifact is 1.47 GB against a 1 GB guideline and GitHub warns each deploy. Crecket's `scripts/sync-tiles.mjs` moves tiles to a Cloudflare R2 bucket and out of the repo; adopt it when convenient. Also makes clones and builds much faster.
- **Keep `main` synced with upstream** (`git fetch upstream && git merge upstream/main`), and rebase `site` on it periodically.
- **Deprecation**: upstream's workflow pins Node 20 actions; GitHub is moving runners to Node 24. Harmless today; bump when upstream does.
- **Discord**: tell Apollyon the fork is live at artywithfriends.com with attribution, and that group/voice work will come as PRs. Mention Crecket's issue #5 so nobody duplicates effort.

---

## Dropped from the old plan

Ads, donations, Buy Me a Coffee, privacy/consent banner, third-party web host, our own calculator/MIL model, our own target-history pane, our own room protocol.
