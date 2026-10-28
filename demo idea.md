# EsKrow+ Onstage Demo Idea

Status: built in `demo-stage/index.html` (open in Chrome or Edge, works offline). Press ? in the demo for presenter keys.

## Core idea

One story on one screen, told as **a scam that gets caught**. Watching a scam fail is more memorable than a normal purchase succeeding, and it shows the security features to the judges (track: Cyber Security & Digital Trust).

## Format

A 16:9 web page (1920×1080, full screen), split in two:

- **Left: phone mockup** of the K PLUS EsKrow+ flow. What the buyer sees.
- **Right: "behind the scenes" panel.** Risk engine scoring the seller, escrow status, and the audit log adding entries live. What the judges need to see.

One person taps through the phone. The presenter advances the seller's side and system events with a key press, so timing always matches the talk.

## Script (about 4–5 minutes)

| Beat | On screen | Talking point |
|---|---|---|
| 1. A deal too good to be true (20s) | Ploy, 24, a first jobber, sees an iPhone for ฿18,000 on IG | "THB 63 billion lost. This is how it starts." |
| 2. Create deal (20s) | Paste seller link/account; deal form already filled in | "Ploy doesn't send money to a stranger. She opens EsKrow+." |
| 3. Risk check (30s) | Account is 9 days old, receives money from many sources, matches a fraud-registry pattern → **High risk** warning | Behavioral Risk Signal |
| 4. Escrow (20s) | Ploy goes ahead anyway; the money is **held**, not sent | "Even if she ignores the warning, the money stays protected." |
| 5. Fake tracking (40s) | Seller uploads a tracking number → check fails (created before the deal, goes to another province) → **release blocked**, refund starts | Verify Tracking: the main moment |
| 6. The rogue admin (90s) | Five steps: a bribed Operations Officer logs in from an unknown laptop → force release hits **four walls** (Zero-Trust, RBAC, Policy, Dual Approval) → an accomplice's approval is rejected ("rules beat signatures") → he edits the log and the **hash chain turns red** → both accounts locked, investigation opened. Ploy's phone shows "asleep" throughout | Zero-Trust Admin, RBAC, Policy-Based Authorization, Dual Approval, Tamper-Evident Log |
| 7. Close (10s) | Ploy sees "฿18,000 refunded" | "Scam stopped before it cost her anything." |

Optional, if time allows: a 15-second legitimate deal first (tracking checks out, funds released), to show normal sellers aren't punished.

## What to build

- A **single local web app** (HTML/JS or React). All data hard-coded; the flow plays out the same way every run.
- **Presenter keys:** → next event, R reset, a hidden key to skip to beat 5.
- **Mocked integrations, labelled honestly:** "Thailand Post / Flash / Kerry API (simulated)", "fraud-registry check (simulated)".
- **A real hash-chained audit log** (SHA-256 in the browser), so the red chain break is real, not an animation.
- K PLUS-inspired styling (green, same card layout), labelled "concept prototype". Keep it local; don't publish it publicly under KBank branding.

## Stage reliability

- Run **offline** from a laptop; no venue wifi.
- **Record a backup video** of the full run; keep it open in a tab.
- No live typing, no login screens. All forms already filled in.
- Large cursor or tap indicator so the audience sees the taps.
- Rehearse with one person speaking and one person driving.

## Q&A backup slide

- Legal basis: the bank is licensed under the Escrow Act B.E. 2551 (see `EsKrow+_Legal_Feasibility.pdf`).
- How release works: the seller sends a shipment ID within 72 hours; the bank checks it with the carrier; only a verified ID reaches the buyer, who confirms with one tap and the seller is paid. No reply in 48 hours pays the seller. A buyer report before payment keeps the money held, and carrier records decide.
- "What if the scammer says they did ship?" They had 72 hours to send a shipment ID the carrier confirms, and were told when theirs failed. After the deadline the contract says refund. ม.๒๓ lets the contract set this rule, so arguing alone cannot freeze the money. Other disputes still freeze funds until the parties agree or a court rules.
- "What if the parcel never arrives after the buyer confirms?" The seller is paid at confirmation, so escrow no longer covers it. The buyer only ever confirms carrier-verified data, and a report goes to the fraud team, which can have the seller's account suspended.
- Refunds go only to the account that paid. A refund paid by mistake can be claimed back from the buyer (Civil and Commercial Code ม.๔๐๖).
- Product quality, wrong or damaged items are out of MVP scope.

## Decisions taken when building

- Length: about 4–5 minutes, 14 steps across 7 beats (beat 6 expanded to 5 steps).
- Legitimate-deal opener: not included (can be added later).
- Release rule: buyer confirms a bank-verified shipment ID (or 48 h silence) and the seller is paid at confirmation, not delivery. No seller appeal after the 72-hour deadline.
- Signed QR: included in beat 4 as "seller sends a fake QR → app blocks it". The live, interactive version is `poc-qr-signing/`.