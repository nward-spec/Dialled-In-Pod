# HANDOFF — The Open 2026 Winner's Bag: Ryan Fox (start a NEW session)

Written 2026-07-19. Purpose: a fresh session (started **after** Network access is set
to **Full**) executes this WITB carousel end-to-end with **real online product images**.

## Why a new session
This project's images must come from the web (manufacturer / GolfWRX hero shots).
The prior session ran under the **Trusted** egress policy, which blocked every image
host (403). We are switching the environment to **Network access → Full**. That change
only applies to sessions started *after* it — hence a fresh session.

**First thing to verify in the new session:** run a quick reachability check before
building, e.g. `curl -sS -o /tmp/t -w "%{http_code}" -L https://www.srixon.com/ ` (or any
product host). If it returns 200-ish, egress is open. If it still 403s, the Full setting
didn't take effect for this session — stop and tell the user. Also re-verify connectors:
list connectors and confirm Slack + Metricool show `enabledInChat: true`.

## The task
Build a 6-frame Instagram carousel (1080×1350) — Ernest Performance × Dialled In Pod
"Winner's Bag" — for **Ryan Fox, 2026 Open Championship champion**. Deliver as **PNG
frames AND a Claude Artifact** ("also in claude design"). Then Slack for approval, and
**only on explicit approval**, schedule via Metricool.

## Club images — READ THIS
- **Use real online hero shots of Fox's actual equipment** (Srixon / Cleveland / Ping /
  Fujikura). Source from manufacturer product pages or the GolfWRX WITB gallery.
- **Do NOT use** the Ernest CGI render set in Google Drive (`assets/ernest-renders/`,
  the `01–09 …-fitting.png` files). The user reviewed them and rejected them ("terrible").
  They are gitignored working copies only.
- Embed images as **base64 data URIs** — required for the Claude Artifact (strict CSP
  blocks external hosts) and keeps the local render self-contained.

## Ryan Fox — verified WITB (2026 Open Championship)
Sources: Golf Monthly, GolfWRX, Today's Golfer, Dunlop/Srixon Tour. Cross-check against
the GolfWRX gallery once egress is open; flag any discrepancy at the Slack step.

| Slot | Club | Detail |
|---|---|---|
| Driver | **Srixon ZXi RKT LS+** | 10.5° · Fujikura Ventus Black 6 X |
| 3-wood | **Srixon ZXi RKT** | 15° · Fujikura Ventus Black 7 X |
| Irons | **Srixon ZXi5** (4i) + **ZXi7** (5–PW) | True Temper Dynamic Gold Tour Issue X100 |
| Wedges | **Cleveland RTZ Tour Rack** | 50° (DG TI X100), 56° & 60° (DG TI S400) |
| Putter | **Ping Anser 2.0** (blade) | one source: PLD Anser 2D prototype — confirm |
| Ball | **Srixon Z-Star XV** | |

Notable talking point: Fox carded a **62 in round 3** (reported as the third player to
shoot 62 at The Open). Winning score / venue not yet verified — confirm before using.

## Brand style (Ernest Performance)
- Near-black background `#0A0908`, champagne-gold accent (~`#C6A15B`), off-white text.
- Premium, minimal, spec-forward. Big winner name + kicker "Winner's Bag".
- Cover shows **3 complete clubs** (not 3 components of one). Wedges/putter frame is a
  grid — watch it for overflow (documented bug in CLAUDE.md).
- Reference layout ("Nicholas Ward personal brand design system") lives in Drive / a
  Claude Project, NOT reachable this session and NOT a published artifact. If the user
  wants a pixel match, get that reference image from them via Drive; otherwise build in
  the brand style above.

## Suggested 6 frames
1. Cover — Ryan Fox · The Open Championship 2026 · Winner's Bag · Ernest × Dialled In Pod
2. Off the tee — driver + 3-wood
3. Irons — ZXi5/ZXi7
4. Wedges & Putter — Cleveland RTZ ×3 + Ping Anser
5. The Ball & the numbers — Z-Star XV + a talking point/stat
6. Outro / CTA — @ernestperformance · @dialledinpod

## Tooling already in the repo (branch `claude/ernest-witb-handoff-jqd7wf`)
- `tools/render_frames.js` — `node tools/render_frames.js <build.html> out/frames`
  (uses pre-installed Chromium; screenshots each `.frame` to 1080×1350 PNG).
- `builds/2026-07-05-john-deere-gotterup.html` — layout/CSS reference (ignore its SVG
  clubs; swap in the real product photos as `<img>` data URIs).
- `CLAUDE.md` — full weekly runbook, the club-alignment history, the egress notes.
- Publish the finished build as a Claude Artifact for the "claude design" deliverable.

## Definition of done
6 frames rendered + reviewed (look at the pixels), Claude Artifact published, posted to
Slack for approval. Commit + push every step (this branch) so nothing is lost. Metricool
only after the user explicitly approves.
