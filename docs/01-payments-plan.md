# 01 — Payments Plan: Live AI Santa Video Calls

*Prepared 2026-11-05. Companion to `04-market-research.md`. Tags: **[V]** = verified from a primary source (regulator, rule text, vendor docs or announcement). **[S]** = secondary source. **[U]** = unverified, or a judgment call. **This is not legal advice. Have a children's-privacy lawyer review §5 before launch.***

---

## 0. Decisions

| Decision | Recommendation |
|---|---|
| Distribution | **Web-first, mobile-optimized web app (PWA).** No native app or IAP for season 1. |
| Processor | **Stripe**: Checkout (or Payment Element), with Apple Pay, Google Pay, and Link; Stripe Tax; Radar. |
| Launch geography | **US only for the Dec 2026 launch.** UK in phase 2, after VAT registration and AADC review. EU later, through a merchant of record (MoR). |
| Pricing model | One-time purchases with no auto-renewal. Three call tiers plus add-ons, gift cards, and a 3-call "North Pole Pass". |
| COPPA consent | Use the **card payment as the verifiable parental consent (VPC) method** (16 CFR 312.5(b)(2)(ii), as amended in 2025), plus a direct notice and an explicit consent checkbox. Gifted calls require a separate parent VPC at redemption. |
| Risk | Contact Stripe this week with a volume forecast. Enable 3DS and Radar rules. Keep session evidence for disputes. Use an automated redo-or-refund policy for technical failures. |

---

## 1. Pricing

### 1.1 Competitive anchors (from market research)

| Segment | Price band |
|---|---|
| AI voice or minute packs (SantaCall, Call Santa Live, CallSantaTonight) | $4–$15 for 2–15 min |
| PNP "Talk to Santa" (AI voice) | £14.99 [V/S]; single PNP call $5.99–$11.99; passes $24.99–$34.99 |
| Live human Santa on Zoom | $15–$25 for 10 min; $35–$50 for 25–30 min; premium $75–$250 |
| JingleRing (human, with keepsake) | from $24.95 |
| Cameo pre-recorded human Santa | $20–$110 |

Our product is live, photoreal video, personalized by the parent, with a keepsake. That puts it **above the AI apps and at or below human Santas**.

### 1.2 Recommended price list (US, season 1)

| SKU | What's included | Price | Notes |
|---|---|---|---|
| **Quick Hello** | 5-min live call, 1 child, personalization form | **$14.99** | Entry tier, priced against PNP Talk to Santa (~$19 equivalent) |
| **Magic Visit** *(hero product)* | 10-min call, up to 2 kids, **keepsake video included** | **$29.99** | Matches the human-Santa 10-min price with better availability. Expect 55–65% of orders [U]. |
| **Family Visit** | 15-min call, up to 4 kids by name, keepsake included | **$44.99** | Siblings and cousins at grandma's house |
| **North Pole Pass** | 3 Magic Visits across the season (e.g., Dec 1 check-in, mid-Dec, Christmas Eve), up to 4 kids, keepsakes included | **$69.99** | One-time payment, not a subscription. Santa remembers the earlier calls *within the season only*. |
| Add-on: Keepsake for Quick Hello | HD recording | **+$6.99** | |
| Add-on: **Christmas Eve slot** | Dec 24, 12:00–21:00 local | **+$10** | Scarcity pricing; capped capacity |
| Add-on: Christmas-morning video message | 30–45 s personalized "I made it! Did you find…" clip | **+$4.99** | Generated asynchronously; low marginal cost |
| Add-on: Nice List certificate and letter (PDF) | Printable | **+$2.99**, or bundled free in Family Visit | |
| Gift card / gift call | Any SKU, sent by email on a chosen date | Same price | Purchaser ≠ parent; see the consent rules in §5 |
| Early-bird (Nov 27–Dec 3) | 20% off all SKUs | — | Black Friday/Cyber Monday booking spike |

**Overruns:** no per-minute overage charges. Santa wraps up gracefully at the time limit, with up to 60 s of grace. A paid "5 more minutes with Santa" extension ($4.99) could be offered *to the parent only* on their control strip. This is optional for v1 [U].

**Subscription for multiple seasons:** **not in v1.** Surprise renewals are PNP's #1 Trustpilot complaint, and the regulatory burden is heavy (FTC negative-option and click-to-cancel rules, plus state automatic-renewal laws such as California's). Instead offer a free "Reserve next year" opt-in with an email reminder in November 2027.

---

## 2. Payments stack

### 2.1 Options compared

| Option | Fees (headline) | Tax handling | Time to launch | Fit |
|---|---|---|---|---|
| **Stripe (direct)**: Checkout or Payment Element, Apple Pay, Google Pay, Link, Radar, Stripe Tax | ~2.9% + 30¢ US cards [S]; Stripe Tax ~0.5% [S]; Radar for Fraud Teams ~2–7¢/txn [U] | We are the seller. Stripe Tax calculates and files; we register where required | **Days** (an existing account is ideal) | **Best.** Fastest, cheapest; Apple Pay/Google Pay with no app; full control over refunds, receipts, and session gating |
| **Stripe Managed Payments** (Stripe acting as MoR) | Reported ~3.5% on top of card fees, or 7–9% all-in [U, conflicting] | Stripe is the legal seller in 80+ countries | GA since Apr 29, 2026 [V]. Eligibility may be narrow [S]; **only works with Checkout and Payment Links** [S] | Good for **UK/EU phase 2**. Confirm product eligibility for a kids' AI service |
| **Paddle** | 5% + 50¢ [S] | Full MoR (VAT, sales tax) | Seller and domain review takes days to weeks [U] | Viable for global, but costlier, less control, and onboarding risk in November |
| **Lemon Squeezy** (owned by Stripe since 2024) | 5% + 50¢ (+1.5% intl) [S] | MoR | Aimed at SaaS and downloads; roadmap uncertain under Stripe [S] | Not recommended |
| **Apple/Google IAP** (native app) | 15–30% commission | Store handles tax | **App review in December is slow.** Apple warns of delays around Dec 20–26 and advises submitting early [S]. A new app in a sensitive kids' category risks rejection | **Avoid for season 1** |
| **Native app + external purchase link (US)** | 0% to Apple *for now*: Apple barred from commissions on US link-outs after the Apr 2025 contempt ruling. The 9th Circuit (Dec 2025) allows a future "reasonable" fee; SCOTUS petition pending [S] | — | Still needs app review | Season 2 option if a native app is needed |

### 2.2 Recommendation

**Use Stripe directly, behind a web app (PWA).**
- WebRTC video runs in mobile Safari and Chrome, so no app is needed.
- Apple Pay and Google Pay work in Stripe Checkout and the Payment Element.
- Link gives 1-click checkout to returning Stripe users.
- Turn on **Stripe Tax**, and register only where we cross state economic-nexus thresholds. Some US states do not tax this kind of digital service; Stripe Tax handles the taxability rules.
- For **UK (phase 2)**: a non-UK seller of B2C electronically supplied services has **no VAT threshold and must register from the first sale** (20% VAT) [S]. An *automated* AI call likely counts as an "electronically supplied service", whereas a live human performer would not [U, confirm with a VAT advisor]. Either register with HMRC and use Stripe Tax, or switch UK/EU checkout to **Stripe Managed Payments** if the account is eligible.

Implementation notes:
- Use **Checkout Sessions** for v1, the fastest route with hosted, PCI-light pages. Put `booking_id`, `sku`, `child_count`, and `slot_id` in `metadata`.
- Fulfil orders from **webhooks only** (`checkout.session.completed`, `payment_intent.succeeded`, `charge.refunded`, `charge.dispute.created`), never from the redirect. Make handlers idempotent on `event.id`.
- Hold capacity for scheduled slots: reserve the slot for 15 minutes when Checkout opens (`expires_at`), then confirm it on the webhook.
- Gift cards: sell them as a Stripe product and issue our own redemption code. Do not use Stripe Customer Balance for kids' gifts.
- Statement descriptor: a recognisable name, e.g. `SANTALIVE*VIDEO CALL`, to cut "unrecognised charge" disputes.

---

## 3. End-to-end flow

```
[Landing] → choose SKU / slot (or "Santa's free now")
   → [Stripe Checkout]  (parent = cardholder; email captured; 3DS when Radar asks)
   → webhook: payment_intent.succeeded
        → create Payment, Entitlement(status=active), Booking(status=confirmed)
        → record COPPA VPC (method=card_transaction, stripe_charge_id, timestamp)
        → email receipt + direct notice + onboarding link
   → [Parent onboarding] (account via magic link)
        → child profiles (name+pronunciation, age, siblings, pets, achievements,
          gentle-nudge, wishlist w/ "OK to promise?" flags, topics to avoid)
        → consent toggles: camera on/off for child, keepsake recording on/off,
          transcript retention, AI disclosure acknowledgement
        → device check (camera/mic/bandwidth test, 20 s)
   → reminders: T-24h, T-1h, T-10m (email/SMS) with join link
   → [Join] parent opens link → server checks entitlement → mints session token
   → [Live call] Santa ↔ child; parent control strip
   → [Post-call] keepsake processed → parent download link (expires 30 days)
        → optional upsell: Christmas-morning message, certificate
        → auto-delete raw media per retention policy
```

### 3.1 Tying a paid entitlement to a time-limited realtime session

- **Entitlement** = a paid right to N seconds of Santa time for a booking.
  - Fields: `seconds_purchased`, `seconds_consumed`, `grace_seconds` (default 60), `window_start`, `window_end`, `status` (`active | in_use | consumed | refunded | expired`).
- **Join endpoint** (`POST /sessions/:booking_id/join`), authenticated by the parent's magic-link session. The server checks all of:
  1. `payment.status = succeeded`, with no refund or dispute.
  2. `consent.vpc_status = granted` for every child on the booking.
  3. `now ∈ [window_start − 10 min, window_end]` for scheduled calls. Instant calls get a 24-hour window from purchase.
  4. `seconds_consumed < seconds_purchased + grace_seconds`.
  5. No other live session exists for this entitlement (single concurrency).
- If all checks pass, the server mints a **short-lived signed token** (JWT, 5–10 min TTL) for the realtime room (LiveKit, Daily, or a custom WebRTC SFU). Claims: `session_id`, `entitlement_id`, `max_end_at = now + remaining_seconds + grace`, and `role=family`. The room server refuses the token after its TTL, but an established connection is governed by the server-side timer, not by the TTL.
- **Metering:** the realtime orchestrator, not the client, emits `heartbeat` every 5 seconds. A ledger row (`session_segments`) records start and end. `seconds_consumed` is the sum of segments in which **Santa was actually rendering to a connected client**, so it does not count time while the child's side is disconnected.
- **Overrun handling:**
  - At remaining = 90 s, the orchestrator tells the Santa agent to "begin a warm goodbye".
  - At 0 s it starts a hard wrap-up with the grace period.
  - At grace end, the server closes the room.
  - If the parent buys an extension, it is added to `seconds_purchased`.
- **Dropped calls and auto-rejoin:**
  - The client auto-reconnects with exponential backoff for up to 60 s and re-calls `/join`, which mints a new token. The entitlement and Santa's conversation context (held server-side, keyed by `session_id`) persist, so Santa resumes with "Oh, the North Pole wind blew our connection away! Where were we…".
  - After 60 s the parent sees a "Rejoin" button. It stays valid until `window_end`, or for 2 hours for instant calls.
- **Automatic technical-failure refund or redo:** a session is classed as *failed* if any of these hold:
  - Santa-side render errors.
  - Total connected time < 50% of purchased.
  - More than 3 reconnects.
  - p95 latency above threshold for more than 30% of the call.

  A failed session triggers an email: "Sorry. Choose a free redo or a full refund." **No support ticket is needed.** Refunds run via the Stripe Refunds API, and the entitlement moves to `refunded`. Failures caused by the client network (e.g., the child's network was offline throughout) get a redo, not a refund.

### 3.2 Refund policy (publish it plainly)

| Situation | Policy |
|---|---|
| Cancel ≥ 2 h before a scheduled slot | Full refund, or reschedule (1-click) |
| Cancel < 2 h before, or no-show | Reschedule once free within 7 days (before Dec 24). Otherwise no refund. *Christmas Eve slots: reschedule only, no refund less than 24 h before* |
| Technical failure on our side | Automatic redo or full refund (see above) |
| "Child didn't enjoy it" within 24 h | Goodwill refund for the first request per account ("Magic Guarantee"). Cheap compared with a $15 dispute fee and the reputational cost |
| Unused gift card | Refundable to the purchaser for 30 days; then credit valid until Jan 31, 2027 (check state gift-card laws [U]) |
| Disputes / chargebacks | Respond with evidence: receipt, VPC record, join logs, session duration, IP and device, keepsake link |

---

## 4. Fraud, chargebacks, and Stripe risk

- **Get pre-approved now.** Contact Stripe (support, or sales if expected volume exceeds ~$250K) with:
  - Business description and the product's kid-safety posture.
  - Expected volume curve: ramp from Nov 27, peak Dec 18–24.
  - Expected average order value (~$30) and refund policy.

  Ask about payout schedule, reserves, and whether Stripe will pre-clear the spike. Third-party sources say notifying Stripe before a seasonal surge helps avoid holds, and that new accounts have a 7–14 day first-payout delay. Both are unverified [U]. **Fund launch costs (GPU, marketing) from capital, not December payouts. Assume payouts could be held until January.**
- **Expected risk profile:**
  - Low order value and digital delivery attract **card testing**: bots trying stolen cards on the $14.99 SKU.
  - **Friendly fraud** (a spouse doesn't recognise the charge, or a parent claims the call didn't happen).
  - Gift cards are a classic vehicle for stolen-card cash-out.
- **Controls:**
  - Radar with rules: block if the CVC or postal code check fails; force 3DS for amounts > $40 and for gift cards; limit to 3 purchases per card per day and 5 per IP per hour.
  - Add Turnstile, hCaptcha, or similar on Checkout entry.
  - Delay gift-card delivery by 1–2 h when the Radar risk score is elevated.
  - Clear statement descriptor; immediate receipt; T-1h reminder (a proven dispute reducer).
  - Keep the evidence bundle above for every session; auto-submit it to Stripe dispute evidence via the API.
  - Watch the dispute ratio daily. Card networks penalise sustained ratios near or above ~0.9–1% (Visa/Mastercard monitoring programs; exact 2026 thresholds unverified [U]).
- **Refunds over disputes:** a refund costs us the order. A dispute costs the order plus ~$15 plus ratio damage. Be generous.

---

## 5. Compliance

### 5.1 COPPA (US): this service is plainly "directed to children"

We collect personal information from children under 13: name, the **child's image and voice in the video and audio stream**, the wishlist, and possibly persistent identifiers. COPPA applies in full. The **amended COPPA Rule** was published Apr 22, 2025, took effect Jun 23, 2025, and has been **fully enforceable since Apr 22, 2026** [V].

**Can the card payment serve as VPC? Yes, if designed correctly.** The amended § 312.5(b)(2)(ii) permits "requiring a parent, in connection with a transaction, to use a credit card, debit card, or other online payment system that provides notification of each discrete transaction to the primary account holder" [V]. The 2025 amendment **removed the word "monetary"**, so a $0 card verification also qualifies [V]. Our paid booking is a real transaction, so it qualifies more cleanly still.

Requirements to make it hold up:
1. **Direct notice to the parent before collection.** Send it in Checkout or immediately after payment, *before* any child data is entered. It states what we collect from the child (name, voice, video image, wishlist), why, which third parties process it (AI and realtime vendors), retention, and parental rights.
2. **Explicit consent step**: a checkbox ("I am the parent/guardian of the child(ren) and consent to…"), linked to the Stripe charge ID in a `consents` table.
3. **The cardholder must be the parent.** For **gift purchases** (grandparent pays), the payment is *not* the parent's. At redemption the parent must provide their own VPC: a $0 card verification via a Stripe SetupIntent, now valid under the "non-monetary" change, or another approved method.
4. **Prepaid or gift cards and some wallets** may not notify a "primary account holder" in the way the rule contemplates [U]. Consider treating prepaid cards (Stripe `card.funding = prepaid`) as insufficient for VPC and asking for a second method.
5. **Separate consent for third-party disclosures not integral to the service**, which the 2025 amendments added [S]. Our AI and realtime vendors are service providers integral to the service, so they are fine under contract. **Never use child data for advertising or model training.** Require zero data retention or no-training terms in vendor DPAs.
6. **Written data retention policy** (now required) and a **written information security program** [S]. Proposed retention:
   - Raw audio and video: deleted at call end, unless a keepsake was opted into.
   - Keepsake: 30 days, then deleted.
   - Transcripts: 30 days, for safety review and disputes.
   - Child profile: until Jan 31, 2027, unless the parent opts to keep it for next year.
7. **Parental rights UI**: review, delete, and revoke consent from the parent dashboard.
8. The **audio-file exception** (a voice used only to answer a request and deleted immediately) does **not** cover us, because we also process video and personalization data. Do not rely on it.

### 5.2 AI disclosure vs. "magic": the central legal and product tension

- **California SB 243** (companion chatbots, in effect Jan 1, 2026) requires disclosure that responses are AI-generated, specific protections for known minors, and break reminders. It carries a **private right of action ($1,000 per violation)** [S].
  - Its definition of a "companion chatbot" may or may not cover a one-off, scheduled character call with no cross-session relationship [U]. The North Pole Pass memory feature brings us closer to the definition.
- **EU AI Act Art. 50** (in force since Aug 2, 2026; not delayed by the Digital Omnibus) requires informing people that they are interacting with AI unless it is obvious [S]. This is a further reason to defer the EU.
- **Recommended posture:**
  - Disclose clearly and repeatedly to the **parent**, who is the purchaser and account holder and is present on the parent device: in marketing ("AI Santa"), at Checkout, and on the parent control strip.
  - Show a small, persistent, kid-unobtrusive "AI" indicator on the child-facing screen.
  - Script Santa never to claim to be human if a child sincerely asks. Pre-approved deflection: "I'm Santa's special magic video helper from the North Pole!"
  - **Get counsel sign-off on this script for California.** If counsel says SB 243 applies, geo-adjust the behavior for CA users.

### 5.3 UK and EU (phase 2)

- **UK GDPR + Age Appropriate Design Code (Children's Code)** apply to any service likely to be accessed by UK children. Requirements:
  - A DPIA.
  - High-privacy defaults.
  - Data minimisation.
  - No nudge techniques.
  - Parental consent for under-13s under UK GDPR Art. 8.

  The ICO is actively enforcing: Reddit was fined £14.47M and Imgur £247K over children's data [S].
- **EU GDPR Art. 8:** the age of digital consent varies from 13 to 16 by member state, plus the AI Act transparency rules above. Use an MoR for VAT, and do a DPIA per market.
- **Recommendation:** launch **US-only**. Geo-block UK and EU checkout in v1 using Radar rules on card or billing country plus IP geolocation. Start the UK DPIA in January for the 2027 season.

### 5.4 Sales tax and VAT

- **US:** Stripe Tax monitors economic nexus (most states use a $100K-sales or 200-transactions threshold, though some states have dropped the transaction count). The taxability of "digital services" or "digital entertainment" varies by state [U]. Register when Stripe Tax flags a threshold.
- **UK:** register from the first sale (no threshold for non-established sellers), 20% [S].
- **EU:** OSS registration, or use an MoR.

### 5.5 Other

- **Terms of Service** that disclose AI, pricing, the refund policy, and the gift-card terms.
- **FTC "Made in USA" / truth-in-advertising:** avoid claims such as "kids can't tell it's AI" (competitors use them, and they invite FTC scrutiny).
- **Accessibility:** captions on the parent screen.
- **Recording consent:** keepsake recording is opt-in by the parent. If the call includes other adults (grandparents on the same device), the consent note covers "everyone in the room" [U]. Check two-party-consent states.

---

## 6. Data model sketch

```sql
-- Parents / account holders (the only "users" who log in)
users(id, email, email_verified_at, name, phone, country, state, created_at, deleted_at)

-- Children: minimal, parent-entered, short retention
children(id, user_id FK, first_name, pronunciation, age_years, pronouns,
         profile_json,            -- siblings, pets, achievements, nudge, avoid-topics
         retain_until, created_at, deleted_at)

wishlist_items(id, child_id FK, label, ok_to_promise BOOL, priority)

products(id, sku, name, base_seconds, max_children, includes_keepsake BOOL, price_cents, currency, active)

-- An order may contain several line items (calls + add-ons) and be a gift
orders(id, purchaser_user_id FK, is_gift BOOL, gift_recipient_email, gift_send_at,
       stripe_checkout_session_id, subtotal_cents, tax_cents, total_cents, currency,
       status ENUM(pending, paid, partially_refunded, refunded, disputed), created_at)
order_items(id, order_id FK, product_id FK, qty, unit_price_cents, addon_of_item_id NULL)

payments(id, order_id FK, stripe_payment_intent_id, stripe_charge_id, amount_cents,
         card_funding ENUM(credit,debit,prepaid,unknown), card_country, wallet,
         radar_risk_level, status, created_at)
refunds(id, payment_id FK, stripe_refund_id, amount_cents, reason ENUM(customer, tech_failure, goodwill, fraud), created_at)
disputes(id, payment_id FK, stripe_dispute_id, reason, status, evidence_submitted_at)

gift_codes(id, order_id FK, code_hash, redeemed_by_user_id NULL, redeemed_at, expires_at)

-- COPPA verifiable parental consent, one row per parent per child (or per booking)
consents(id, user_id FK, child_id FK,
         method ENUM(card_transaction, card_verification_0, other),
         stripe_ref,                 -- charge or setup_intent id
         notice_version, scope_json, -- camera, recording, transcript retention
         granted_at, revoked_at, ip, user_agent)

-- What the parent is allowed to consume
entitlements(id, order_item_id FK, user_id FK,
             seconds_purchased, seconds_consumed, grace_seconds,
             max_children, includes_keepsake BOOL,
             status ENUM(active, in_use, consumed, refunded, expired),
             valid_from, valid_until)

-- A booked (or instant) slot
bookings(id, entitlement_id FK, scheduled_start, scheduled_end, tz, is_christmas_eve BOOL,
         type ENUM(scheduled, instant), status ENUM(held, confirmed, completed, no_show, cancelled, failed),
         reminder_state_json)
booking_children(booking_id FK, child_id FK)

-- One live room instance; several per booking after reconnects
sessions(id, booking_id FK, room_id, started_at, ended_at,
         end_reason ENUM(completed, timeout, parent_ended, safety_stop, tech_failure),
         quality_json,               -- p95 latency, reconnects, render errors
         failure_classification NULL)
session_segments(id, session_id FK, connected_at, disconnected_at, santa_rendering_seconds)
join_tokens(id, session_id FK, jti, issued_at, expires_at, revoked_at)  -- audit only

recordings(id, session_id FK, storage_key, status, delete_after)  -- keepsake only if consented
transcripts(id, session_id FK, storage_key, delete_after)
safety_events(id, session_id FK, type, severity, excerpt_redacted, created_at)

webhook_events(id, stripe_event_id UNIQUE, type, processed_at)  -- idempotency
```

Key invariants:
- A booking can start only if its entitlement is `active`, a payment has `succeeded` without being refunded or disputed, and **every** `booking_children` row has an unrevoked `consents` row.
- `seconds_consumed` is written only by the orchestrator, from `session_segments`.
- Deletion jobs run nightly against `retain_until` and `delete_after`.

---

## 7. Launch checklist (payments and compliance)

| By | Item |
|---|---|
| Nov 9 | Stripe account in production; contact Stripe risk/support with the forecast; enable Apple Pay domain verification, Google Pay, and Link |
| Nov 12 | Products and prices in Stripe; Checkout plus webhooks; Radar rules; statement descriptor |
| Nov 16 | COPPA direct notice, privacy policy, ToS, and refund policy reviewed by counsel; AI-disclosure script approved |
| Nov 20 | Entitlement and join-token service; metering; auto-failure classification; refund automation |
| Nov 24 | Load test of the checkout and join paths at 10× forecast peak; card-testing protection on |
| **Nov 27** | **Open pre-booking (BF/CM early-bird)** |
| Dec 1 | Calls live |
| Daily in Dec | Dispute and refund dashboard; Radar review; slot-capacity monitoring (Christmas Eve cap) |

---

## Sources

- COPPA amended rule, Federal Register (Apr 22, 2025): https://www.federalregister.gov/documents/2025/04/22/2025-05904/childrens-online-privacy-protection-rule
- Venable on removal of "monetary" from the payment VPC method: https://www.venable.com/insights/publications/2025/01/ftc-finalizes-coppa-rule-changes-in-the-biden
- Kelley Drye on the FAQ H.5 update: https://www.kelleydrye.com/viewpoints/blogs/ad-law-access/ftc-updates-coppa-faqs-on-parental-consent-mechanisms/
- Securiti summary of the 2025 COPPA amendments: https://securiti.ai/ftc-coppa-final-rule-amendments/
- COPPA in full effect (Apr 22, 2026): https://www.finnegan.com/en/insights/articles/coppas-amended-rule-is-now-in-full-effect-what-operators-need-to-know.html ; https://www.toyassociation.org/ta/PressRoom2/News/2026-News/updated-coppa-rule-requirements-take-effect-april-22.aspx ; https://www.hunton.com/privacy-and-information-security-law/ftc-publishes-final-coppa-rule-amendments
- California SB 243 and Character.AI: https://calawyers.org/privacy-law/regulatory-focus-on-ai-companion-character-chatbots/ ; https://xident.io/blog/companion-ai-age-verification-sb-243-state-law-wave-2026/ ; https://www.cnbc.com/make-it/2025/11/24/characterai-to-ban-teens-from-open-ended-chats-human-interaction-is-crucial-psychotherapist-says.html
- EU AI Act Art. 50 timing after the Digital Omnibus: https://www.morganlewis.com/blogs/sourcingatmorganlewis/2026/08/eu-ai-acts-transparency-rules-what-went-into-effect-on-2-august ; https://www.joneswalker.com/en/insights/blogs/ai-law-blog/yes-august-2-still-matters-the-eu-approved-a-high-risk-ai-delay-but-most-trans.html ; https://www.gibsondunn.com/eu-ai-act-omnibus-agreement-postponed-high-risk-deadlines-and-other-key-changes/
- UK ICO Children's Code enforcement: https://www.edtechinnovationhub.com/news/ico-fines-reddit-1447m-over-childrens-data-failures-and-weak-age-checks ; https://5rightsfoundation.com/the-age-appropriate-design-code-can-protect-children-from-ai-harms-if-properly-enforced/ ; https://www.mondaq.com/uk/privacy-protection/1805106/ico-updates-guidance-on-using-childrens-information
- UK VAT on digital services for non-established sellers: https://quaderno.io/guides/united-kingdom/vat/registration/ ; https://www.fonoa.com/resources/country-tax-guides/united-kingdom/tax-on-digital-services
- Stripe Managed Payments GA (Apr 2026): https://x.com/stripe/status/2049532106712453307 ; https://docs.stripe.com/changelog/dahlia/2026-04-22/managed-payments?locale=en-GB ; limitations: https://freemius.com/blog/stripe-merchant-of-record/ ; https://fungies.io/stripe-managed-payments-vs-merchant-of-record-2026/ ; https://tiun.io/blog/cost-of-stripe-managed-payments-2026
- Stripe fees: https://www.nerdwallet.com/business/software/learn/stripe-fees ; https://dodopayments.com/blogs/stripe-fees-calculator
- Paddle vs Lemon Squeezy: https://resources.rework.com/tools/billing-revenue/paddle-vs-lemon-squeezy ; https://dev.to/jettfu/stripe-vs-paddle-vs-lemon-squeezy-fee-comparison-2026-2c77
- Apple external links and Epic: https://www.macrumors.com/2025/12/11/apple-app-store-fees-external-payment-links/ ; https://www.revenuecat.com/blog/growth/apple-anti-steering-ruling-monetization-strategy ; https://finance.biggo.com/news/202605010055_Apple_Loses_Stay_in_Epic_App_Store_Fee_Case
- Apple holiday review slowdown: https://www.macrumors.com/2024/12/02/app-store-review-times-holidays/ ; https://airship.com/?p=1062
- Stripe holds and seasonal spikes (third-party): https://www.chargeblast.com/blog/why-payout-holds-happen-how-to-avoid-them ; https://www.chargeflow.io/blog/stripe-payout-hold-what-to-do-when-stripe-freezes-your-payouts ; https://www.disputifier.com/post/why-stripe-and-shopify-hold-funds-and-how-to-avoid-payout-delays
- Competitor prices: see `04-market-research.md` §Sources (PNP, JingleRing, SantaCall, Call Santa Live, human Santas, Cameo).
- PNP billing complaints (subscription lesson): https://www.trustpilot.com/review/portablenorthpole.com
