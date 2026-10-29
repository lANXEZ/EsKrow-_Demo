# EsKrow+ System Design

Status: agreed design as of 29 Oct 2026. Merges our earlier design with the accepted parts of "DealSure — ออกแบบระบบใหม่ v2" (see `DealSure_v2_Differences.pdf` for the item numbers U1–U16 and M1–M11 used below).

Target users: First Jobbers aged 20–29, the age group that loses most to online scams (28% of victims), buying through social commerce (M11).

## Deal flow

1. **Seller creates the deal (M2).** The seller enters the item, price, buyer and the buyer's shipping address. The bank first checks the seller's account status in its core banking system (U3). The core system already freezes accounts flagged through AOC 1441 / the Central Fraud Registry, so a frozen account can't open a deal; EsKrow+ has no separate AOC integration. Every deal gets a new deal ID and its own digital contract.
2. **Bank-signed deal QR.** The bank signs the deal QR (ECDSA P-256, Thai QR/EMVCo layout, M9). The seller sends it in chat. The buyer's bank app verifies the signature with the bank's pinned public key before showing anything. An ordinary PromptPay QR gets a "Not a protected deal" warning (M10).
3. **Buyer check (U4).** The buyer's account is checked too: not frozen, not a mule, no history of dispute abuse. Pass/fail only, and the result is never shown to the seller (PDPA).
4. **Risk check (M7), two layers:**
   1. **Account status from core banking:** frozen by an authority order (AOC 1441 / CFR) means Blocked. This catches accounts already reported.
   2. **Account and transaction analysis:** account details (age, KYC) and how money moves through the account. This catches mule accounts nobody has reported yet.

   **Every deal is held in escrow**, whatever the result (decided 29 Oct; replaces M3, where green sellers were paid instantly). The risk result only changes what the buyer is told:

   | Level | What the buyer sees |
   |---|---|
   | Normal | No notice |
   | Things to know | An amber card of plain facts about the account (e.g. "Account opened less than 1 month ago", "No completed deals yet", "Reported by buyers this week"). The pay button counts down 5 s so the facts get read; "Not now" sits below it |
   | Blocked | "This deal can't go ahead right now", with no reason. Only for an account frozen by an authority order |

   **Facts, not verdicts** (agreed 29 Oct), so a seller with an old, resolved record is not branded for life:
   - Account status is a gate only. The reason for a freeze is never shown to the buyer.
   - The card states facts about the account, never labels about the person ("high risk", "suspicious"), and never the score or thresholds.
   - Every notice ends with: "Whichever seller you choose, your money is held by the bank until you confirm the parcel."
   - Signals expire after a clean period, completed deals build visible positive history, and sellers can appeal (admin queue, U13).

   The card also shows the seller's partly masked real name, and any stats the seller chose to share (see Seller stats and reputation).
5. **Contract and payment (M4).** The seller and the buyer accept the contract. **Both sides do a live face scan on every deal** (replaces the step-up rule in U5), so the person behind each account is verified and can be traced. The buyer pays into the bank's segregated escrow account under the Escrow Act B.E. 2551.
6. **Shipment proof (M5).** The seller ships the goods and submits the carrier's shipment ID within **48 hours** of payment. The system does **not** check it with the carrier: the mentor asked to keep third-party integrations to a minimum. The ID is passed to the buyer as entered.
7. **Contract expiry.** No shipment ID within 48 h: the contract expires and the full amount is refunded to the paying account. To trade again, the two sides create a new contract.
8. **Payout check.** Before any payout, EsKrow+ re-reads the seller's account status. If the account was frozen while the deal was running, the money is not released: it stays held and follows the authority's order.
9. **Buyer checks and confirms.** The app notifies the buyer. She checks the shipment details for that ID with the carrier herself (carrier app or website): right address, right recipient. If it looks right, she taps **Confirm** and the money is paid to the seller immediately. If she neither confirms nor reports within **24 h** of receiving the ID, the money is paid to the seller.
10. **Report (M5 + M6).** A small **Report** button, before payment. The buyer must give a reason and a photo (for example a screenshot of the carrier's tracking page).
   1. The deal moves to DISPUTED and the money stays held.
   2. The two sides can settle in the app: cancel, partial refund or reship. Agreement ends the dispute.
   3. If they can't agree, the money stays held for up to about 7 days, and an Evidence Pack (PDF + QR) goes to AOC 1441 / police (U12).
   4. After that, any state freeze order applies. With no order, the money stays frozen until the parties agree or a court rules (ม.๒๓).

   The bank never judges who is right, and no staff member reviews the shipment.

### Deal states

```
CREATED ─paid─► HELD ─shipment ID sent─► SHIPPED ─confirm / 24 h silence─► RELEASED
                HELD ─no shipment ID in 48 h─► EXPIRED (refund)
                                               SHIPPED ─report─► DISPUTED ─agreed─► MUTUAL (cancel / partial refund / reship)
                                                                 DISPUTED ─no agreement─► REPORTED (held, Evidence Pack → freeze order or court)
```

## Seller stats and reputation (added 29 Oct)

Sellers see all their own stats in a seller dashboard and choose which ones to show buyers as reputation.

- **Dashboard (seller only):** total money received through completed EsKrow+ deals, completed deals, unique buyers, time selling on EsKrow+, plus the less flattering numbers (expired contracts, disputes) so the seller sees the full picture.
- **Opt-in display:** the seller ticks any of the positive stats to show. They appear on the buyer's seller-check page, beside the bank's facts, under "Stats shared by the seller · verified by the bank". The seller can turn each one off at any time (consent under the PDPA).
- **Only completed EsKrow+ deals count.** Raw account inflow is never used: a mule account receiving victims' transfers would otherwise look like a top seller.
- **The bank computes every number.** The seller can choose what to show but can't edit it. Money is shown rounded (e.g. "฿1.2M+") so it doesn't expose exact income.
- **Unique buyers** are distinct face-verified people, and the graph checks (U15) discount circles of accounts trading with each other, so the count can't be inflated with fake buyers.
- **Reputation sits beside the facts, never replaces them.** The bank's facts (account age, recent buyer reports) still show whatever the seller picks. Good stats can outweigh a warning in the buyer's eyes, but they can't hide one.

## Bank staff (U13)

- No one can move deal money alone. Every action needs two approvers from Risk or Compliance, never the requester's own team.
- Admins work only from registered, bank-managed devices with a hardware key (Zero-Trust).
- Role-based access (RBAC): Operations can view deals but not release money.
- The policy engine sits on top of all of this. An expired contract or an open dispute cannot be overridden by any number of approvals.
- Admin work: the threat queue (fake-QR scans, mule patterns), appeals from wrongly rated sellers, forwarding cases to authorities.
- Every action goes into the audit log.

## Evidence and audit log (U11)

- Every event is append-only.
- Each deal has its own SHA-256 hash chain.
- A Merkle root over all chains is signed by the bank's HSM on a regular schedule and stored in WORM (write-once) storage.
- One record can be proven without opening the rest.

## Architecture (U8–U10, U14)

- App module inside the bank app → API gateway (token check, rate limit, WAF, idempotency key) → five services:

  | Service | What it does | Backed by |
  |---|---|---|
  | Listing/QR | Signs deal QRs | HSM |
  | Deal | State machine and deadlines | Postgres |
  | Risk | Reads scores from cache in milliseconds | Redis |
  | Payment | Transfer / hold / refund through the existing transfer system, with idempotency keys | Existing transfer system |
  | Shipment | Records the seller's shipment ID and starts the buyer's 24 h window | No carrier integration |

- Kafka carries events to background work: the Evidence service, the risk pipeline, Case/Report and the Admin console.
- **Risk pipeline:** rules + XGBoost + GraphSAGE on the transfer graph, computed in the background.
- **Account status:** read from the core banking system at deal creation and again before payout. No external government API in phase 1.
- **Fail-safe:** if the Risk service is down, show "Things to know: the seller check isn't available right now", never "Normal". The deal is held in escrow either way.
- **Identity:** a live face scan by both parties on every deal. This makes renting or selling a bank account to a scammer (a mule account) much harder, and ties every deal to a verified person.
- **Device protection:** app attestation (Play Integrity / App Attest), and token-bucket rate limits per user, device and QR.

## Scope

- **Phase 1 is on-us only (U1):** buyer and seller both bank with the same bank, because the risk check needs the seller's transaction history.
- **Phase 2:** cross-bank, using federated learning + MPC (multi-party computation).
- **Phase 2 idea, not in the MVP:** a police check limited to two yes/no answers (active fraud case? fraud conviction in the last N years?), never shown to buyers. Needs a data-sharing agreement with the Royal Thai Police and a legal basis under PDPA ม.๒๖.
- **MVP covers:**
  - fake or high-risk sellers
  - payment with no shipment (contract expires, automatic refund)
  - fake shipment claims (caught by the buyer before confirming, then Report)
- **Out of MVP scope:** product quality, and wrong or damaged items. These go through Report → settle → Evidence Pack, not a bank decision.

## On hold

- U2: buyer-side "Request a bank-backed deal" link.
- U6: seller packing photo or video.
- U7: reusable per-item listing QR with sold status.
- M1: renaming (EsKrow+ / K PLUS kept for now). The DealSure doc says the competition bans KBank / KBTG / K PLUS names and logos in the product, prototype and demo.

## Open questions

- 48 h shipping and 24 h confirmation windows: confirm they fit carrier pickup times, long weekends, and a buyer who is busy for a day.
- Confirm with the bank that its core system already freezes accounts flagged through AOC 1441 / CFR, and how quickly.
- How long a clean period clears an old signal (e.g. 6 or 12 months).
- Hold period for an unresolved dispute (about 7 days) before the Evidence Pack and freeze-until-court stage.
