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
| 1. A deal too good to be true (20s) | Ploy, 24, a first jobber, sees an iPhone for ฿20,000 on IG | "THB 63 billion lost. This is how it starts." |
| 2. The deal QR (30s) | Seller first sends an ordinary PromptPay QR → app warns "Not a protected deal". Ploy asks for an EsKrow+ deal; the seller creates one (item, price, Ploy's name and address) and sends the **bank-signed deal QR** → verified on her phone | Bank-signed QR |
| 3. Risk check (30s) | Account is 9 days old, receives money from many sources, matches a fraud-registry pattern → **Red** card with reason categories (no score). Main button "Don't pay yet"; "Pay anyway" counts down | Behavioral Risk Signal |
| 4. Escrow (20s) | Ploy pays anyway. Every deal is held in escrow: contract signed by the seller and Ploy (both by live face scan) and the bank; the money is **held**, not sent | "Even if she ignores the warning, the money stays protected." |
| 5. Fake shipment (50s) | Seller sends a shipment ID. Ploy looks it up in the carrier's app: wrong recipient, Chiang Mai, created before the deal → taps **Report** (reason + screenshot) → money stays held. Seller refuses to settle → **Evidence Pack to AOC 1441** | Buyer check + one-tap Report: the main moment |
| 6. The rogue admin (90s) | Five steps: a bribed Operations Officer logs in from an unknown laptop → force release hits **four walls** (Zero-Trust, RBAC, Policy, Dual Approval) → an accomplice's approval is rejected ("rules beat signatures") → he edits the log and the **hash chain turns red** → both accounts locked, investigation opened. Ploy's phone shows "asleep" throughout | Zero-Trust Admin, RBAC, Policy-Based Authorization, Dual Approval, Tamper-Evident Log |
| 7. Close (10s) | AOC case → authority order → Ploy sees "฿20,000 refunded" | "Scam stopped before it cost her anything." |

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
- Full rules: `system-design.md`.
- How release works: every deal goes through escrow, whatever the risk level. The seller ships and sends the shipment ID within 48 hours; the app notifies the buyer, who checks the shipment details with the carrier herself (no carrier integration, per the mentor) and confirms with one tap, and the seller is paid. No reply in 24 hours pays the seller. No shipment ID in 48 hours: the deal expires and the buyer is refunded.
- "What if the buyer says the shipment is wrong?" Report (reason + photo) → the money stays held → the two sides settle in the app (cancel, partial refund, reship) → no agreement: Evidence Pack to AOC 1441 / police → frozen until an authority order or a court ruling (ม.๒๓). The bank never judges who is right.
- "What if the seller really shipped?" Then he settles, or shows the authorities his proof. The bank never has to decide.
- "Why doesn't the bank check the tracking number?" Fewer third-party integrations. The contract shows the buyer exactly what to compare, and the decision to pay stays with her.
- "Why not show the risk score?" Reason categories only, so scammers can't tune their accounts to slip under a threshold.
- "What if the parcel never arrives after the buyer confirms?" The seller is paid at confirmation, so escrow no longer covers it. The buyer only ever confirms carrier-verified data, and a report goes to the fraud team, which can have the seller's account suspended.
- Refunds go only to the account that paid. A refund paid by mistake can be claimed back from the buyer (Civil and Commercial Code ม.๔๐๖).
- Product quality, wrong or damaged items are out of MVP scope.

## Decisions taken when building

- Length: about 4–5 minutes, 14 steps across 7 beats (beat 6 expanded to 5 steps).
- Legitimate-deal opener: not included (can be added later).
- Release rule: the buyer checks the shipment ID herself and confirms (or 24 h silence); the seller is paid at confirmation, not delivery. Seller has 48 hours to ship and send an ID; then the contract expires. No seller appeal.
- Signed QR: beat 2. The seller's ordinary QR gets a warning, then the seller sends a bank-signed deal QR. The live, interactive version is `poc-qr-signing/`.
- Timeline: paid 13 Oct 19:45 → fake ID 14 Oct 10:12, reported 10:20 → Evidence Pack 16 Oct → rogue admin 17 Oct 01:50–02:17 → authority order and refund 20 Oct 14:00. Length: 15 steps.