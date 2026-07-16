# Ernest Performance — "Winner's Bag" (WITB) Instagram Carousel

Weekly PGA Tour winner's club breakdown ("What's In The Bag"), published as an
Instagram carousel. Posted as a collab between **Ernest Performance** and the
**Dialled In Pod**.

> **Provenance note:** This tree was reconstructed on 2026-07-16 from a written
> handoff after a prior session's work failed to persist (it was never committed
> to a remote). `CLAUDE.md` and `tools/compose_club_stack.py` are faithful to the
> described architecture. `Winners Bag Carousel.html` is a **skeleton** — it must
> be validated against a real build before the first live post. The original
> product-image assets under `assets/products/` were binary and could not be
> recovered; they are re-sourced per winner as part of the weekly workflow anyway.
> **Everything here is now committed and pushed** so this cannot recur.

---

## The one thing to read before touching club-stack layout

**Do not try to align club heads, shafts, and grips with CSS.** This was
attempted across many prior iterations and it does not work. Here is why, so
nobody re-litigates it:

- Manufacturer product photos of club **heads** are shot at an angle and have a
  short **diagonal shaft stub already baked into the image** at the hosel.
- A separate straight **shaft** image cannot be made to line up with that baked-in
  diagonal stub using CSS transforms — the perspective, the hosel entry angle, and
  the stub length all fight you. You get a visible kink or a double-shaft at the
  hosel every time.

**The fix (current architecture):** pre-render each club into a single, correctly
aligned PNG *before* it ever reaches the HTML. A Python/Pillow compositor
(`tools/compose_club_stack.py`) takes the head, shaft, and grip source images plus
a per-club manifest of anchor points, rotates the shaft to continue the head's
**true hosel axis**, stacks the grip on top, and writes one flattened image to
`assets/products/stack/<club>-stack.png`. The HTML then just places that finished
PNG. No CSS alignment of components. Ever.

If a stack looks wrong, fix the **manifest anchor points** and re-run the
compositor — do not add CSS transforms in the HTML to compensate.

---

## File layout

```
project/
  CLAUDE.md                      # this file
  Winners Bag Carousel.html      # the carousel (6 frames). Uses pre-rendered stacks only.
  tools/
    compose_club_stack.py        # head+shaft+grip -> single aligned PNG
  assets/
    products/
      manifest.json              # per-club source paths + hosel anchor/axis
      raw/                        # source head/shaft/grip images (per winner, re-sourced weekly)
      stack/                      # compositor output: <club>-stack.png  (what the HTML consumes)
  out/                           # rendered/screenshotted frames for Slack approval
```

## The carousel (6 frames)

1. **Cover** — winner + tournament + "Winner's Bag", Ernest × Dialled In Pod
   branding. Shows **3 real, complete clubs** (three separate stacks), NOT three
   components of one club. (An earlier version mistakenly showed head/shaft/grip
   of a single club here — fixed.)
2. Driver + fairway woods
3. Irons
4. **Wedges + putter** — grid layout. Watch this frame: it previously overflowed
   its grid (too many items pushed content past the 1080 boundary). Fixed; keep an
   eye on it whenever the wedge count changes.
5. Ball + specs
6. Outro / CTA

Frames are 1080×1350 (Instagram portrait). Render in headless Chromium and
screenshot each frame to `out/` for approval before posting — do not trust the
markup alone, always look at the rendered pixels.

---

## Weekly workflow (the trigger runs this)

**Cadence:** PGA Tour final rounds finish Sunday; GolfWRX / manufacturer WITB
posts land Monday. A trigger checks **hourly every Sunday from 5:00 PM Eastern
onward** and continues into Monday until a confirmed WITB is available.

**The trigger's prompt should just say "follow the WITB workflow in CLAUDE.md" —
do not duplicate these steps into the trigger.**

### Steps

1. **Confirm a tournament finished** and identify the winner (name, event).
2. **Cross-check the WITB** — pull the winner's What's-In-The-Bag from GolfWRX
   (and/or the manufacturer's WITB post) and verify it matches the confirmed
   winner. Do not proceed on an unconfirmed or mismatched bag.
3. **Source product images** for each club (head / shaft / grip) into
   `assets/products/raw/`, update `assets/products/manifest.json` with anchor
   points, and **run the compositor**:
   ```
   python tools/compose_club_stack.py --all
   ```
   This writes aligned `assets/products/stack/<club>-stack.png` files.
4. **Build the carousel** — fill `Winners Bag Carousel.html` with this week's
   winner, clubs, and specs; render all 6 frames to `out/` and eyeball them.
5. **Post to Slack for approval** — send the rendered frames to the team Slack
   channel and wait. **Do not schedule anything without explicit approval.**
6. **Only on explicit approval:** schedule the post via **Metricool**
   (`createScheduledPost`) with the caption and the 6 frames in order.

### Connectors required

- **Slack** — approval step (post frames, await go-ahead).
- **Metricool** — scheduling the approved carousel.
- **Google Drive** — available fallback for supplying product images in-policy.

Before using any in a session, verify the connector is actually enabled for
*this* session (connector toggles only take effect for sessions started after the
toggle). List connectors and confirm `enabledInChat: true` before relying on them.
Connector traffic is routed through Anthropic, so connectors work **without** any
network-allowlist entry.

### Network access requirement (IMPORTANT — learned the hard way)

The compositor needs real product photos, but the default **Trusted** egress
policy blocks all golf-media and manufacturer image hosts (they return a
`403` policy denial — do NOT retry or route around them). WebSearch still works,
so winner-ID and WITB *text* cross-check are fine; only *image download* is blocked.

To let the pipeline web-source photos, the environment's **Network access** must be
set to **Custom** (with the hosts below + "include default package managers"
checked) or **Full**. Edit this in the Claude Code on-the-web environment settings
(cloud icon → edit environment → Network access). Changes only apply to sessions
started *after* the change. Docs: https://code.claude.com/docs/en/claude-code-on-the-web#network-access

Suggested Custom allowlist (add one per line; brands vary weekly so occasional
additions may be needed — a blocked run will name the host):

```
# golf media / WITB sources
*.golfwrx.com
*.golf.com
*.golfdigest.com
*.todays-golfer.com
*.nationalclubgolfer.com
*.2ndswing.com
# manufacturer sites + image CDNs
*.ping.com
*.taylormadegolf.com
*.titleist.com
*.callawaygolf.com
*.bridgestonegolf.com
*.cobragolf.com
*.mizunogolf.com
*.srixon.com
*.pxg.com
# retailer product-image CDNs (stock most brands)
*.pgatoursuperstore.com
*.golfgalaxy.com
*.tgw.com
cdn.shopify.com
# generic reference images
en.wikipedia.org
upload.wikimedia.org
```

**Full** access avoids weekly host maintenance entirely — the pragmatic choice if
the per-host allowlist becomes a nuisance. Either way, **MCP connectors (Slack /
Metricool / Drive) do not need allowlisting.**

---

## Open items / gotchas

- **Instagram reference post** `https://www.instagram.com/p/Dae4ozSDebh/` was the
  desired final layout reference. `instagram.com` is blocked by the sandbox network
  policy (`host_not_allowed`) — it was never fetched. If the layout needs to match
  it, ask the user for a screenshot rather than retrying the URL.
- **Durability:** this repo now has a remote (`origin`). Commit and push after every
  meaningful change — the prior loss happened because work lived only in an
  ephemeral container with no remote.
