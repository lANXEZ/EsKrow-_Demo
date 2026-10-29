# EsKrow+ System Design

Status: agreed design as of 29 Oct 2026. Merges our earlier design with the accepted parts of "DealSure — ออกแบบระบบใหม่ v2" (see `DealSure_v2_Differences.pdf` for the item numbers U1–U16 and M1–M11 used below).

Target users: First Jobbers aged 20–29, the age group that loses most to online scams (28% of victims), buying through social commerce (M11).

## Deal flow

1. **Seller creates the deal (M2).** The seller enters the item, price, buyer and the buyer's shipping address. The bank first checks the seller: account not frozen, not on the mule list (U3). Every deal gets a new deal ID and its own digital contract.
2. **Bank-signed deal QR.** The bank signs the deal QR (ECDSA P-256, Thai QR/EMVCo layout, M9). The seller sends it in chat. The buyer's bank app verifies the signature with the bank's pinned public key before showing anything. An ordinary PromptPay QR gets a "Not a protected deal" warning (M10).
3. **Buyer check (U4).** The buyer's account is checked too: not frozen, not a mule, no history of dispute abuse. Pass/fail only, and the result is never shown to the seller (PDPA).
4. **Risk check (M3, M7).** The risk card shows a level and reason categories, never the score or thresholds:

   | Level | Main button | Payment |
   |---|---|---|
   | Green | "Accept and pay" | Paid to the seller immediately, no escrow |
   | Yellow | "Accept and pay", short warning | Escrow if the amount is unusual for this buyer |
   | Red | "Don't pay yet"; "Pay anyway" is secondary, after a countdown | Escrow always |
   | Prohibited | No pay button | None (e.g. account frozen by law) |

   The card also shows the seller's partly masked real name, account age as a range, and completed deals with unique counterparties.
5. **Contract and payment (M4).** For escrow deals, the seller and the buyer accept the contract. The buyer confirms with step-up authentication: face scan the first time, on a new device or for a large amount, otherwise PIN (U5). The buyer pays into the bank's segregated escrow account under the Escrow Act B.E. 2551.
6. **Shipment proof (M5).** The seller ships the goods and submits the carrier's shipment ID within **72 hours** of payment. The system does **not** check it with the carrier: the mentor asked to keep third-party integrations to a minimum. The ID is passed to the buyer as entered.
7. **Contract expiry.** No shipment ID within 72 h: the contract expires and the full amount is refunded to the paying account. To trade again, the two sides create a new contract.
8. **Buyer checks and confirms.** The app notifies the buyer. She checks the shipment details for that ID with the carrier herself (carrier app or website): right address, right recipient. If it looks right, she taps **Confirm** and the money is paid to the seller immediately. If she neither confirms nor reports within **48 h** of receiving the ID, the money is paid to the seller.
9. **Report (M5 + M6).** A small **Report** button, before payment. The buyer must give a reason and a photo (for example a screenshot of the carrier's tracking page).
   1. The deal moves to DISPUTED and the money stays held.
   2. The two sides can settle in the app: cancel, partial refund or reship. Agreement ends the dispute.
   3. If they can't agree, the money stays held for up to about 7 days, and an Evidence Pack (PDF + QR) goes to AOC 1441 / police (U12).
   4. After that, any state freeze order applies. With no order, the money stays frozen until the parties agree or a court rules (ม.๒๓).

   The bank never judges who is right, and no staff member reviews the shipment.

### Deal states

```
CREATED ─green─────────────────────────────────────────► RELEASED
CREATED ─yellow/red─► HELD ─shipment ID sent─► SHIPPED ─confirm / 48 h silence─► RELEASED
                      HELD ─no shipment ID in 72 h─► EXPIRED (refund)
                                               SHIPPED ─report─► DISPUTED ─agreed─► MUTUAL (cancel / partial refund / reship)
                                                                 DISPUTED ─no agreement─► REPORTED (held, Evidence Pack → freeze order or court)
```

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
  | Shipment | Records the seller's shipment ID and starts the buyer's 48 h window | No carrier integration |

- Kafka carries events to background work: the Evidence service, the risk pipeline, Case/Report and the Admin console.
- **Risk pipeline:** rules + XGBoost + GraphSAGE on the transfer graph, computed in the background. External checks (police / AOC 1441 / CFR) run over OAuth2 + mTLS, cached about 24 h, behind a circuit breaker.
- **Fail-safe:** if the Risk service is down, show yellow ("can't check right now"), never green.
- **Device protection:** app attestation (Play Integrity / App Attest), and token-bucket rate limits per user, device and QR.

## Scope

- **Phase 1 is on-us only (U1):** buyer and seller both bank with the same bank, because the risk check needs the seller's transaction history.
- **Phase 2:** cross-bank, using federated learning + MPC (multi-party computation).
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

- The external risk checks (police / AOC 1441 / CFR APIs) are also third-party calls. Confirm with the mentor that these are acceptable, since carrier checks were dropped for that reason.
- 72 h shipping and 48 h confirmation windows: confirm they fit carrier pickup times and long weekends.
- Hold period for an unresolved dispute (about 7 days) before the Evidence Pack and freeze-until-court stage.
- Which government APIs can really be called (CFR, AOC 1441), in real time or in batches.
