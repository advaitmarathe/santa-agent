# Santa Agent — Plan Overview

A paid, real-time, photoreal AI Santa that children (ages 3–10) talk to live from a parent's phone or laptop. Launch target: early December 2026. The only technical constraint is **latency**.

This page summarises the decisions. The detail, the vendor comparisons and the sources are in:

| Doc | Covers |
|---|---|
| [`docs/01-payments-plan.md`](docs/01-payments-plan.md) | Pricing, Stripe setup, how a payment unlocks a call, COPPA consent, refunds, data model |
| [`docs/02-avatar-plan.md`](docs/02-avatar-plan.md) | Avatar vendor comparison, building the Santa likeness, interruptions, Christmas Eve scaling |
| [`docs/03-agent-plan.md`](docs/03-agent-plan.md) | Cascade vs speech-to-speech, voice stack, Santa persona, guardrails, fallbacks |
| [`docs/04-market-research.md`](docs/04-market-research.md) | Competitors, what parents want and fear, demand timing, positioning |

Most latency figures are vendor claims. Each doc marks what it could not verify.

---

## The key decisions

### 1. Voice: a cascade pipeline, not speech-to-speech
**Use a streaming cascade:** speech-to-text → LLM → text-to-speech. Don't use OpenAI GPT-Live or Gemini Live end to end.

- **A custom Santa voice.** The speech-to-speech models either don't offer custom voices, restrict them, or forbid imitating characters. ElevenLabs and Cartesia let us clone a hired Santa voice actor.
- **Guardrails.** A cascade lets us check every phrase *before* Santa says it, which is non-negotiable for 3–10 year-olds. Speech-to-speech can't do this.
- **Small latency cost.** GPT-Live claims about 0.8 s. Our cascade targets about 0.85 s (p50) to first audio.
- **Plan B:** if the week-1 test shows audio-native models understand small children much better, use one as the "ears + brain" that outputs text. Our own voice and safety check stay in place.

| Stage | Choice | Hot failover |
|---|---|---|
| Transport | LiveKit Agents, all stages in AWS us-east-1 | — |
| Speech-to-text + turn detection | Deepgram Flux | AssemblyAI Universal / Cartesia Ink |
| LLM | Claude Haiku 5.5, thinking off, prompt caching, sent to Anthropic API and Bedrock at once | Gemini Flash-class |
| Voice | ElevenLabs Turbo/Flash, Santa actor clone | Cartesia Sonic, clone of the same actor |

### 2. Avatar: Tavus primary, Anam hot fallback. Decart is out
- **Decart** retired its avatar and lip-sync products in May 2026. Its Lucy model only transforms a live camera feed, which would need a human Santa on every call.
- **Primary: Tavus Phoenix-4.5** with a custom Santa face. It claims about 134 ms from audio to video and has built-in listening behaviour.
- **Fallback: Anam Cara-4.1.** It had the best measured connection time and has an explicit stop command for interruptions.
- **Overflow: HeyGen LiveAvatar.**
- **How it connects:** the avatar only receives Santa's generated audio and never sees or hears the child. That matters for privacy rules.
- **Building Santa:** film a professional Santa with a real beard once, on a North Pole set. The footage trains every vendor. He needs a likeness contract reviewed by a lawyer.
- ⚠️ Tavus also runs its own free consumer AI Santa, so it is a supplier and a competitor at once. Factor that into the bake-off and the contract (data use, exclusivity of our Santa face).

### 3. Payments: Stripe, web app, one-time prices
- **Stack:** Stripe Checkout with Apple Pay, Google Pay and Link, plus Stripe Tax and Radar, behind a phone-friendly web app (PWA).
  - US-only at launch.
  - No App Store or Google Play purchases this season: they take a 15–30% cut and December app review is slow.
- **Prices:** one-time, no subscription.

  | Product | Price |
  |---|---|
  | Quick Hello: 5 min | $14.99 |
  | **Magic Visit (flagship): 10 min, 2 kids, keepsake recording** | **$29.99** |
  | Family Visit: 15 min, 4 kids | $44.99 |
  | North Pole Pass: 3 visits | $69.99 |
  | Christmas Eve slot | +$10 |

- **Consent:** the card payment doubles as verifiable parental consent under COPPA. A grandparent's gift purchase doesn't count, so the parent consents separately when they redeem it.
- **Failures:** any call that fails for technical reasons gets an automatic redo or refund.
- **Calls are booked time slots, not on-demand.** This lets us reserve avatar and voice capacity for Dec 18–24.

### 4. Market position
- **Competitors:**
  - Portable North Pole is the incumbent: cheap pre-made videos plus an AI voice-only call, with reviews split by billing complaints.
  - Tavus offers a free 3-minute AI video Santa that feels laggy and admits it's an AI.
  - Live human Santas on Zoom charge $15–$250.
- **Our edge:**
  - The most realistic live Santa.
  - Deep personalisation from the parent's form.
  - Visible safety controls for parents: a live transcript and a kill switch.
  - Honest one-time pricing and reliability at the booked time.
- **Demand:** booking spikes on Black Friday (Nov 27) and Cyber Monday. Calls peak Dec 18–24.

---

## Expected latency (child stops talking → Santa responds)

The two plans give slightly different totals: the avatar plan expects about 0.8 s and the voice plan about 1.1 s at p50. Plan for the more conservative numbers.

| Stage | Target |
|---|---|
| Mic capture + network | ~130 ms |
| Turn detection (end of child's speech) | ~260 ms |
| LLM first token | ~300 ms |
| Safety check on first phrase | ~15–35 ms |
| Voice first audio | ~120 ms |
| **Santa audible** | **p50 ≤ 850 ms, p95 ≤ 1.4 s** |
| Avatar lip-sync | 150–250 ms |
| **Santa's lips moving** | **p50 ≈ 1.1 s, p95 ≤ 1.8 s** |
| Kid interrupts Santa → Santa stops | ≤ 200 ms (no vendor publishes this; must measure) |

**Hiding the remaining delay:**
- A cached "Ho ho… hmm" nod/acknowledgement makes Santa visibly react within about 0.5–0.6 s.
- A "Connecting to the North Pole…" animation covers connection time.

---

## Biggest risks
1. **Understanding small children.** Off-the-shelf speech recognition makes very many errors on preschoolers (published figures as high as 70%+). Week-1 test on consented recordings of staff families' children.
2. **Capacity on Christmas Eve.** Get written concurrency reservations from Tavus and Anam for Dec 20–26.
3. **Breaking the Santa magic, or the opposite: AI disclosure law.** California SB 243 and the EU AI Act may require disclosing that the user is talking to an AI.
   - Plan: disclose clearly to the parent.
   - The parent picks "magic" or "gently honest" mode for the child.
   - Santa never claims to be human.
   - A lawyer must sign off.
4. **A child discloses abuse.** Escalation when the parent may be involved needs counsel and a child-safety advisor.
5. **Stripe payout holds** during the December spike. Pre-clear volume with Stripe now.
6. **Real latency worse than vendor claims.** Every number above is gated on our own measurement.

---

## Timeline (from today, Nov 5)

| Dates | Milestone |
|---|---|
| Nov 5–7 | Hire Santa actor (likeness + voice contract via counsel). Contact Stripe with a volume forecast. Open enterprise talks with Tavus, Anam, ElevenLabs and Cartesia. |
| Nov 9–13 | Two bake-offs: speech-to-text on child speech; avatars (Tavus, Anam, HeyGen) on measured end-to-end p95 latency, interruption stop time and parent-panel realism ratings. Film the Santa footage and record the voice. |
| **Nov 14** | **Lock vendors.** |
| Nov 14–24 | Build the full pipeline, guardrails, parent dashboard and Stripe booking flow. Run safety red-teaming. |
| Nov 25 | Closed beta with friends and family. |
| **Nov 27** | **Public booking opens (Black Friday).** Calls start ~Dec 1. |
| Dec 18–24 | Peak. War room on call; failover drills done beforehand. |

## Decisions needed from you
- How Santa answers "Are you real?" (magic vs gently honest default) — with legal.
- Escalation policy for distress disclosures.
- US-only launch, or add UK/EU (adds VAT, the UK Children's Code and EU AI Act work)?
- Approve hiring a Santa actor (photoreal likeness) over a fully AI-generated Santa face.
