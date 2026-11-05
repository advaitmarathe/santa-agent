# 02 — Avatar Plan: Real-Time Photoreal Santa

*Research date: 2026-11-05. Launch target: early December 2026. Cost is not a constraint; latency is the only hard one.*

> **How to read the numbers.** Almost every latency figure below is published by the vendor itself, and vendors measure different things: render time per frame, audio-to-first-frame, connection time, or a full turn. Treat them as **vendor claims** unless marked *measured*. The only multi-vendor measurement found was run by Anam, a competitor of the others, which says it has a conflict of interest. **No figure here replaces our own bake-off (§7).**

---

## 1. TL;DR

| | Choice | Why |
|---|---|---|
| **Primary** | **Tavus Phoenix-4.5**: custom Santa face trained from footage of a real actor, run in **LiveKit mode** (our agent owns speech-to-text, the LLM, TTS and safety; Tavus only renders) | Fastest audio→video figure claimed by any vendor (134 ms); built-in active listening and micro-expressions; emotion control; a first-party LiveKit plugin; a custom face from video. |
| **Hot fallback** | **Anam Cara-4.1 Express**: custom Santa avatar made from the same footage, behind the same LiveKit abstraction | Best *measured* connection time (median 1.42 s, P95 2.97 s) and turn latency (median 1.24 s, full pipeline) in the only cross-vendor benchmark; audio-passthrough mode with explicit `interruptPersona()`; 25 fps at 1152×768, portrait supported. |
| **Overflow / tertiary** | **HeyGen LiveAvatar**, "Avatar Only" (bring-your-own voice stack) | Advertises "unlimited concurrency" on paid plans plus dedicated enterprise capacity. Useful if Tavus or Anam capacity runs out on Christmas Eve. |
| **Not recommended for launch** | Decart, Unreal MetaHuman + Pixel Streaming, NVIDIA ACE, Runway Characters, Hedra, Simli, D-ID, Synthesia, bitHuman | Reasons in §3. The most important: **Decart no longer serves an avatar or lip-sync model.** It retired *Live Avatar* and *LipSync Live* in May 2026. |

**Avatar-stage latency budget:** **≤ 250 ms** from the first TTS audio byte to the first lip-synced frame on the child's screen (render about 130–180 ms plus jitter buffer and WebRTC). For the full turn (child stops talking → Santa's lips move), **about 700–900 ms is realistic at p50 and about 1.3 s at p95** (§6).

**Decision gate:** run the bake-off (§7) from Nov 6 to Nov 13 and lock the primary vendor by **Nov 14**. Ship with both vendors integrated and switchable for each session.

---

## 2. What the avatar layer must do (requirements)

1. **Photoreal Santa with a consistent identity**: a real white beard, a red suit, a North Pole workshop behind him, and the same face on every call.
2. **Audio-driven, streaming lip sync.** Audio comes in from our TTS (or a speech-to-speech model) and the video frames must stay tight to it (A/V offset of 45 ms or less is the broadcast norm to aim for).
3. **Alive while listening**: blinking, nodding, small smiles and gaze toward the camera while the child talks. A frozen Santa breaks the magic for 3–10-year-olds right away.
4. **Instant interruption.** When the child talks over Santa, video and audio must stop within about 200 ms, and Santa should return to a listening pose without a visual "pop".
5. **WebRTC to mobile browsers.** Most parents will use iOS Safari or Android Chrome on a phone, often held upright in portrait.
6. **Burst scaling**: thousands of concurrent sessions from Dec 20 to Dec 24, peaking on Christmas Eve evening across US time zones.
7. **Child-safe data flow.** Under the amended COPPA rule (compliance date 2026-04-22), the avatar vendor should ideally **never receive the child's audio or video**, only Santa's synthesized audio.

---

## 3. Vendor landscape (as of 2026-11-05)

### 3.1 Decart, investigated in depth as requested

**Bottom line: Decart does not offer a usable audio-driven avatar product today.**

- **History.** In Oct 2025, Decart launched a **Lip Sync API** ("stream audio, get synced video frames") and later an **Avatar Live** model (Jan 2026). It published a demo with ElevenLabs and Pipecat.
- **Retirement.** Decart's changelog says that **in May 2026 "LipSync Live, Live Avatar, Lucy Motion, Lucy v1 Realtime, and Lucy Restyle v1 Realtime" stopped being served.** No dedicated avatar or lip-sync model appears in the current model table.
- **What Decart sells now**, all **video-to-video** models:
  - **Lucy 2.6** (Oct 2026): real-time editing of a live camera stream, including character swap from a reference image, background swap and style changes. It runs at 1280×720, landscape or portrait, over WebRTC. Docs give no fps or latency figure; third-party reports cite 30 fps.
  - **Pricing:** $0.02/s ($1.20/min) standard; $0.04/s ($2.40/min) in "fast mode", which lowers latency. No concurrency limit is published; enterprise pricing is on request.
  - **Audio input:** not mentioned anywhere in the docs.
  - **Other models:** Lucy 2.5, Lucy Restyle 2, the Lucy VTON try-on models and Oasis 3 (a world model).
- **Could Decart still play a role?** Lucy 2.6 needs a *driving video*. Two ways to use it:
  - (a) **A live human Santa performer** on camera, transformed into "perfect Santa". This would be photoreal with perfect lip sync, but it does not scale: one performer serves one call.
  - (b) **Chain it after another avatar's output** to "Santa-fy" it. This adds a second WebRTC hop and unknown latency, doubles the cost, and risks lip and identity drift.

  Neither fits a sub-1 s, thousands-of-concurrent-calls product. **Not recommended for launch.** Possible R&D later: a premium "live human Santa" tier that uses Lucy 2.6 with fast mode to make volunteer performers look consistent. *Unverified*: Lucy 2.6 latency, fps under load, and behaviour on a fast-talking mouth.

### 3.2 Comparison table

Legend: **V** = vendor claim, **M** = measured by a third party (Anam's Oct 16, 2026 benchmark, which has a conflict of interest), **U** = could not verify.

| Vendor / model | Realism (subjective) | Avatar-stage latency | Session start (time to first frame) | Lip-sync approach | Custom Santa? | Listening/idle | Transport / LiveKit | Concurrency for a Dec spike | Notes / policy |
|---|---|---|---|---|---|---|---|---|---|
| **Tavus Phoenix-4.5** (Sep 2026) | Top tier. Tavus claims a 2:1 blind-test preference over the nearest competitor (V) | **134 ms audio→video (V)**. The full CVI is advertised as about 1 s utterance-to-utterance, or under 500 ms on the homepage (V) | **Median 5.0 s, P95 9.9 s (M)** | Gaussian-diffusion renderer driven by audio; face, expression, gestures, upper body | **Yes.** Video path (static background, chest-up or waist-up) or image path (preview in about 1 min, tuning takes a few hours). Accepts stylized human faces. Rejects people under 18, public figures and copyrighted characters | **Best in class.** Active listening, emotion control (Phoenix-4+), Sparrow-2 turn-taking (V) | Its own WebRTC (Daily); **LiveKit plugin** (`livekit-plugins-tavus`; Phoenix-4.5 Pro faces recommended); Pipecat; Agora | Self-serve plans are capped (about 3–15 streams; conversation length capped at 10–60 min). **Enterprise concurrency is negotiated (U)** | Green-screen background replacement is **not available with Phoenix-4.5**, so the background is whatever was in the training footage. The customer is responsible for use by minors (ToS); the AUP has zero tolerance for content harming minors |
| **Anam Cara-4.1 Express** (Oct 2026) | Very high. Anam claims "most expressive" (V) | about 150–180 ms server-side (V); **median 1.24 s full turn (M, own pipeline)** | **Median 1.42 s, P95 2.97 s (M)**; 2.76 s when behind LiveKit (M) | Pixel-space diffusion, 25 fps, 1152×768 or 768×1152 | Yes, through Anam Lab or the API. Single-image support claimed by third parties (U) | Good (V); human evaluation included interruptibility | WebRTC; JS and Python SDKs; LiveKit plugin (Python and Node); **audio passthrough (beta)**: PCM16 at 16 kHz with `interruptPersona()` | "Autoscaling"; numbers not published (U) | SOC 2 Type II. No child-specific policy found (U) |
| **HeyGen LiveAvatar** | High (V); 1080p optional (adds latency) | First frame under 300 ms (V); third parties estimate about 1–2 s end to end | **Median 3.66 s, P95 7.6 s (M)** | Proprietary | Yes: 2 min of footage (15 s listening, 90 s talking, 15 s listening) or a single photo. Live or recorded consent | Listening segments come from the training footage | Web SDK. LiveKit plugin (Python). Transport not documented (U) | **"Unlimited concurrency" on paid plans; dedicated enterprise capacity (V)** | Session caps: 20 min on Essential, 60 min on Business. Enterprise above $40k/yr |
| **Synthesia** (interactive, API) | High (stock or personal avatars) | Not published | Median 3.44 s via LiveKit (M) | Proprietary | Personal avatar (enterprise) | U | LiveKit plugin 1.8.8 (Oct 2026) | Enterprise only (U) | Enterprise sales cycle is too slow for Dec 1 (U) |
| **Runway Characters (GWM-1 Avatars)** | High; strong for stylized characters | 37 ms model time per frame (V); **1.75 s server-side turn (V, Runway's own figure)** | U | Generative world model; the paper says "head motion only" | Yes, from a single image (photoreal, cartoon, mascot) | U | LiveKit integration; 24 fps HD | U | About $0.20/min. 1.75 s turn time is too slow |
| **LemonSlice 2.1 Flash** | Good | 471 ms time to first byte (V); 2.04 s end to end (V) | Median 9.7 s, P95 14.4 s (M) | 20B diffusion transformer | Single image | U | LiveKit, Pipecat | U | Too slow |
| **Simli Trinity** | Medium. Reviewers flag circular eye movement and mechanical idle motion | Under 300 ms speech→video (V) | U | Gaussian splatting, about 25 fps | Face from an image | **Weak idle behaviour (reviews)** | LiveKit plugin | U | Very cheap, but idle quality falls short |
| **D-ID V4 Expressive** (Mar 2026) | Medium-high | Model under 120 ms; conversation under 500 ms (V) | U | Proprietary | Photo | U | Agents SDK; LiveKit plugin | 3 embedded agents on Advanced; enterprise above that (V) | Front-end-oriented SDK |
| **Beyond Presence Genesis** | Medium-high | Under 100 ms inference; 250 ms or less globally (V) | U | Proprietary | Yes | U | LiveKit plugin | U | Too little independent data |
| **Hedra Live Avatars** | Medium; one reviewer found it slow and low quality | Under 100 ms at launch (V); about 500 ms to first frame (third party) | U | Character-3 | Image | U | LiveKit (launch partner) | **10-minute session cap; US-only hosting (third party)** | No longer in LiveKit's current avatar plugin list (U) |
| **bitHuman** | Medium (stylized/2.5D) | Under 200 ms on 1–2 CPU cores (V); about 1 s first frame (third party) | U | On-device or cloud | Yes | U | LiveKit plugin | Self-hostable, so scales on CPU | Less photoreal |
| **Newcomers**: Protoface (LiveKit plugin from Jul 2026, $0.01/min), Keyframe Labs, TruGen (about 800 ms), Spatius, Avatario, AvatarTalk, Ojin Oris (under 200 ms, V) | U | U | U | U | U | U | All have LiveKit plugins | U | Too new to bet a Dec 1 launch on |
| **NVIDIA ACE / Audio2Face-3D + Unreal MetaHuman + Pixel Streaming** | High *if* a studio builds a custom MetaHuman Santa with groom hair; strong uncanny-valley risk | Audio2Face about 80 ms (third party, V); UE render plus Pixel Streaming adds tens of ms | Instance boot takes minutes, so it needs warm pools | 3D blendshapes (Audio2Face is open-sourced: SDK under MIT, models under the NVIDIA Open Model License) | Full control: a custom 3D Santa, beard simulation, animated workshop | Fully scriptable idle animation | Pixel Streaming (WebRTC) | **One GPU per session**; must self-provision about 1,000+ GPUs | Months of work for a production-quality photoreal beard. Forum reports of lip-sync and latency problems on UE 5.7 + Convai. **Not achievable by Dec 1** |

**Key observations**

- **Session start ≠ turn latency.** Tavus's slow session start (about 5 s median) can be hidden behind a "Connecting to the North Pole…" intro (§5.4). Turn latency cannot be hidden.
- **Only Tavus and Anam combine top-tier realism, low avatar-stage latency, real listening behaviour, custom faces and a LiveKit plugin.** HeyGen is a close third, and the strongest on concurrency claims.
- **Every serious vendor now ships a LiveKit Agents avatar plugin** (LiveKit lists 16 providers). That makes LiveKit the natural abstraction layer for failover between vendors.

---

## 4. Recommended architecture

### 4.1 Topology

```
 Parent's phone/laptop (browser)                       Our cloud (LiveKit Cloud or self-hosted SFU)
 ┌─────────────────────────────┐   WebRTC (mic)   ┌──────────────────────────────────────────────┐
 │ LiveKit JS client           │ ───────────────▶ │ LiveKit room "call-<id>"                     │
 │  - Santa video track        │                  │                                              │
 │  - Santa audio track        │ ◀─────────────── │  ┌────────────── Agent worker (ours) ─────┐  │
 │  - overlay: snow, frame,    │   WebRTC (A/V)   │  │ VAD + turn detector (child-tuned)       │  │
 │    "Santa is listening"     │                  │  │ STT (streaming) → safety → LLM → TTS    │  │
 └─────────────────────────────┘                  │  │  (or speech-to-speech model)            │  │
                                                  │  │ AvatarSession (Tavus | Anam | HeyGen)   │  │
                                                  │  └───────────┬─────────────────────────────┘  │
                                                  │  PCM audio  │  ▲ lk.clear_buffer / playback   │
                                                  │  (ByteStream│  │ RPCs                          │
                                                  │             ▼  │                               │
                                                  │  ┌── Avatar worker (vendor cloud GPU) ─────┐  │
                                                  │  │ joins room as participant; publishes    │  │
                                                  │  │ synced Santa video + audio on behalf of │  │
                                                  │  │ the agent (lk.publish_on_behalf)        │  │
                                                  │  └─────────────────────────────────────────┘  │
                                                  └──────────────────────────────────────────────┘
```

- **Our agent owns the conversation.** Speech-to-text, the kid-safety layer, the LLM and TTS (or a speech-to-speech model) all run in *our* LiveKit agent. The avatar vendor only receives **Santa's TTS audio** and returns video. The vendor never sees or hears the child. That is a large COPPA and data-minimization win (see §8). It also lets us swap vendors without touching the conversation logic.
- **Use Tavus's LiveKit mode, not its full CVI pipeline.** Tavus says its full pipeline (with Sparrow turn-taking) gives the lowest latency. However, we need our own safety layer and our own child-tuned turn detection, and we do not want the child's audio and camera going to Tavus's perception model (Raven). In the bake-off, *also* measure the full CVI pipeline with a custom LLM endpoint. If it is more than 150 ms faster per turn, revisit the choice with legal input.
- **Audio route.** The agent's audio output goes to the avatar worker through LiveKit's `DataStreamAudioOutput` (a ByteStream of PCM). The worker publishes **audio and video together**, so lip sync is guaranteed by the vendor rather than by us. Never publish TTS audio directly to the room in parallel; it will drift from the video.
- **Vendor abstraction.** Write a small `SantaAvatar` factory that returns a Tavus, Anam or HeyGen `AvatarSession` based on a per-session routing decision (feature flag + health + capacity). All three are LiveKit plugins with the same `AvatarSession.start(session, room)` shape.

### 4.2 Building the Santa character

**Recommendation: hire a professional bearded Santa performer, film him once in a built workshop set, and train every vendor's face from that one master footage. Back it with a fully synthetic Santa image as an emergency path.**

| Option | Pros | Cons | Verdict |
|---|---|---|---|
| **A. Licensed real Santa actor (natural beard)** | The most photoreal result. A real beard moves naturally, which generative models handle better than a costume beard. One footage set feeds Tavus, Anam and HeyGen. Clear consent chain | Requires a digital-replica contract, a shoot day and a fee | **Primary** |
| B. Fully synthetic Santa (AI-generated still image → image-to-face) | Fast (Tavus 4.5 previews in about 1 min); no performer rights | Vendors' consent/subject rules for synthetic people are **unaddressed (U)**. Less natural motion from a single image. Risk of uncanny beard texture | **Emergency backup only** |
| C. 3D MetaHuman Santa | Full control | Months of work; beard and groom realism is hard; one GPU per call | Rejected for 2026 |

**Shoot spec (week of Nov 9)**

- **Performer.** A real, full white beard (no glued beard: costume edges flicker in diffusion output). Not a recognisable celebrity, because Tavus rejects "obvious public figures". Adult performer only; vendors reject subjects under 18.
- **Framing.** Chest-up or waist-up, centered, face fully visible, beard not covering the mouth. Mustache trimmed so the upper lip is visible: lip-sync legibility matters for kids. *Risk:* the Phoenix-4 rules prohibit a "dense beard over the neck"; **Phoenix-4.5 does not list that restriction, but test it on day 1.**
- **Set.** A static, lit North Pole workshop behind him: a warm fireplace glow *without* visible flicker, a tree, toys. With Tavus Phoenix-4.5, green-screen background replacement is **not available**, so the background in the footage *is* the background. Keep it static. A moving fire or twinkle lights would freeze or warp.
- **Footage.**
  - Record about 2–4 minutes per vendor spec. Use HeyGen's structure: 15 s listening, 90 s talking, 15 s listening. Add extra *listening* takes: gentle nods, warm smiles, a "hmm!" lean-in, a twinkle in the eye. These teach the models Santa's idle and listening behaviour.
  - Record **one vertical (9:16) and one horizontal (16:9) take**: mobile portrait matters, and Anam supports 768×1152 portrait.
  - Record a separate **consent statement** clip; Tavus and HeyGen require one.
- **Voice.** The avatar voice is our TTS Santa voice, which is a separate decision. If we clone the same performer's voice, the consent contract must cover voice too.
- **Client-side scene dressing** (adds no latency): a CSS/WebGL overlay with gentle falling snow, a frosted window frame, a "Santa is listening…" twinkle indicator and a name banner. This brings the scene to life without asking the avatar model to animate the background.

**Legal and consent for the actor** (talk to counsel before the shoot)

- Sign a written **digital-replica licence**. It should describe the specific intended uses (interactive AI Santa conversations with children, the brand, the term, territories), give compensation and a renewal path, and require the performer to be represented by a lawyer or union. That representation requirement comes from California AB 2602, which makes vaguely described replica terms unenforceable. New York has a similar 2024 law.
- If the performer is SAG-AFTRA, the 2026 replica terms (for contracts from 2026-07-01) require written consent.
- Track the **federal NO FAKES Act**. The 2026 bill (S. 4591 / H.R. 8915) was reported out of Senate Judiciary on 2026-06-18 and has not been enacted as of Oct 2026. Draft the contract to satisfy it anyway.
- **Exclusivity.** The performer must not appear as Santa for any competing AI product, and we need the right to *retire* the face. Include a morality clause and a takedown process.
- **Vendor terms.** Each vendor (Tavus, HeyGen, Anam) requires the subject's explicit consent clip and keeps rights to it. Get Tavus to confirm in writing that a non-celebrity actor playing Santa (a public-domain folk character) is not rejected as a "copyrighted character" or "public figure" *(U)*.

### 4.3 Listening state behaviour

- **The avatar stream is always on.** From the moment the session starts, the vendor publishes idle or listening video; we never show a black frame. Tavus Phoenix-4 and 4.5 have built-in "active listening" (continuous facial and head motion). HeyGen and Anam draw idle behaviour from the training footage, which is why the extra listening takes in §4.2 matter.
- **Emotion cues.** With Tavus, set emotion through the LiveKit plugin (≥ 1.8.9 Python / 1.9.4 Node). Call `wait_for_join()` first, because emotions set before the avatar joins are dropped. Map LLM intent to emotion: child shares a wish → "joyful"; child is shy → "warm/gentle".
- **Backchannels and "thinking" fill.**
  - When end-of-turn is detected and the LLM is still producing, emit a **pre-synthesized short audio cue** ("Ho ho, hmm…", "Oh my!") within about 300 ms. This makes the avatar *speak* sooner and hides LLM time-to-first-token.
  - Cache these cues as PCM per vendor so they need no TTS round trip.
  - Use them sparingly and randomized: about 30–40% of turns, never twice in a row.
- **UI hint.** Show a subtle client-side "Santa is listening" sparkle when VAD sees child speech. Kids 3–6 benefit from explicit turn cues.

### 4.4 Interruptions (the child talks over Santa)

1. **Detection** happens in *our* agent: VAD plus the LiveKit turn detector on the child's mic.
   - Tune for children: higher-pitched voices, giggles, background siblings, TV noise.
   - Require about 300–400 ms of sustained speech (or ≥ 2 recognized words) before treating it as a barge-in. This avoids interrupting on a cough or "um".
   - Echo cancellation must be on: browser AEC via `getUserMedia({echoCancellation:true})`. Without it, Santa's own voice from the phone speaker triggers barge-ins. **Test on iPhone speakerphone specifically.**
2. **Stop.** The `AgentSession` cancels the LLM and TTS streams and calls `clear_buffer` on the avatar audio output. That sends the `lk.clear_buffer` RPC to the vendor's avatar worker, which drops the queued audio and video and returns to listening.
   - For Anam in passthrough mode, the equivalent is `interruptPersona()` plus `endSequence()`. Anam warns the avatar keeps playing buffered audio without them.
   - **Target: ≤ 200 ms from interruption detection to Santa visibly stopping.** Measure it in the bake-off. *U*: no vendor publishes a stop-latency figure.
3. **False interruption recovery.** Use LiveKit's pause/resume path where the plugin supports it: pause on a possible interruption, then resume if it turns out to be noise. The SpatialReal plugin documents this pattern; verify per vendor.
4. **Graceful yield.** After an interruption, Santa's next turn acknowledges it ("Oh! Tell me, tell me!") instead of restarting the old sentence. Track the transcript of what was *actually played* (LiveKit playback position) so the LLM's context does not include unspoken words.
5. **Keep TTS chunks small** (sentence or clause-level streaming). That keeps the vendor-side buffer short, so there is less queued video to flush.

---

## 5. Scaling for the Christmas Eve spike

1. **Capacity forecast.** Model peak concurrency from the sales forecast: for example, 50k bookings on Dec 24 × 5-minute calls, concentrated in 17:00–20:00 local across 4 US time zones, gives a peak of about **3–5k concurrent sessions** *(illustrative, replace with real numbers)*. Bookings let us shape demand: **sell time slots**, not on-demand calls. This is the single most effective scaling control.
2. **Contracts (this week).**
   - Sign **enterprise agreements with both Tavus and Anam**, each with a *written* concurrency reservation of at least 60% of forecast peak, a Dec 20–26 burst SLA, a named on-call engineer, and the session cap removed (self-serve Tavus caps conversations at 10–60 min).
   - Add HeyGen LiveAvatar (Business or Enterprise; "unlimited concurrency" claimed) as overflow.
   - Ask each vendor for its *GPU region list* and target US-East and US-West.
3. **Routing.**
   - A session router picks the vendor per call from: health (error rate in the last 5 min), live concurrency against the reservation, and the region.
   - Pin each child to one vendor for the whole call; never switch Santa's face mid-call.
   - **Use the same Santa footage for every vendor** so Santa looks similar across them. They will differ slightly, so keep a returning family on the same vendor (store `vendor` on the family profile).
4. **Pre-warm.**
   - Start the avatar session while the parent completes the pre-call screen (consent checkbox, mic permission, "tell Santa your child's name").
   - The 1.4–5 s connection time is hidden behind a 5–8 s **"Connecting to the North Pole…" sleigh animation**. Call `wait_for_join()` before Santa's first line.
5. **Graceful degradation ladder.** Primary vendor → secondary vendor → overflow vendor → **"Santa's Elf waiting room"**: a pre-rendered video of Santa ("I'm feeding the reindeer, I'll be right with you!") with a queue position. Never show an error screen to a child.
6. **LiveKit itself.** Use LiveKit Cloud (or self-host the SFU with autoscaling) and agent workers on autoscaled CPU nodes. The avatar GPUs belong to the vendor. Load-test at 1.5× forecast peak by **Dec 10**, using synthetic agent-to-agent calls.
7. **Session time limits.** Product-side cap of about 10 min, with a warm goodbye at 8 min. This controls cost and frees capacity.

### 5.1 Mobile browser notes (general engineering knowledge; verify in QA)

- WebRTC through the LiveKit JS SDK works on iOS Safari 15+ and Android Chrome. **Autoplay requires a user gesture** ("Tap to call Santa" button) before Santa's audio can play.
- Use **portrait 9:16** framing on phones and landscape on laptops. That means a portrait-trained face per vendor (Anam supports 768×1152 natively; Tavus 4.5 is chest-up; check whether portrait output is supported, *U*).
- Prefer **720p**; 1080p (HeyGen) adds latency. Use simulcast/adaptive streaming and TURN over TLS on port 443 for restrictive home and school networks.
- iOS Low Power Mode and backgrounding the tab pause video. Detect that and pause the agent.

---

## 6. Latency budget

### 6.1 Full turn (child stops talking → Santa's lips move with audio)

| Stage | Target p50 | Target p95 | Notes |
|---|---|---|---|
| Child's audio uplink (device → SFU) | 40 ms | 100 ms | WebRTC/Opus, nearest region |
| End-of-turn detection (VAD silence + turn model) | 200 ms | 350 ms | Kids pause mid-sentence, so don't go below about 180 ms. *Owned by the voice-pipeline doc* |
| Streaming STT finalization | 50 ms | 120 ms | Overlaps with end-of-turn detection |
| Safety check + LLM time-to-first-token | 200 ms | 400 ms | Or a speech-to-speech model; *owned by the voice-pipeline doc* |
| TTS time-to-first-audio-byte | 80 ms | 150 ms | Streaming TTS |
| **Avatar stage** (first PCM byte in → first synced frame out of the vendor) | **150 ms** | **250 ms** | Tavus claims 134 ms, Anam about 150–180 ms (V) |
| Downlink (vendor GPU → SFU → device) + jitter buffer + decode | 60 ms | 150 ms | |
| **Total** | **≈ 780 ms** | **≈ 1.3 s** | A backchannel cue (§4.3) can make Santa *visibly respond* at about 500–600 ms |

### 6.2 Avatar stage, broken down

| Sub-stage | Budget | How we measure it |
|---|---|---|
| Agent → avatar worker audio transfer (LiveKit ByteStream, same region) | ≤ 20 ms | Timestamp in the chunk metadata |
| Vendor buffering before rendering starts | ≤ 40 ms | Ask the vendor for its minimum audio lookahead (*U* for all) |
| Model render of the first frame | ≤ 140 ms | Vendor claim; verify with LiveKit `playback latency` metrics and `wait_playback_start=True` |
| Publish + encode | ≤ 30 ms | |
| **Avatar stage total** | **≤ 230 ms p50 / ≤ 250 ms target** | Glass-to-glass: clap test with a high-speed phone camera (§7) |
| A/V sync offset at the client | ≤ 45 ms | Visual frame vs audio waveform on recordings |
| Interruption stop (barge-in detected → avatar mouth stops) | ≤ 200 ms | Scripted barge-in tests |
| Session start (connect → first frame) | ≤ 3 s p50, ≤ 6 s p95 | Hidden by the pre-call animation |

---

## 7. Bake-off plan (Nov 6 – Nov 13)

1. **Day 1–2.** Create test faces of a stand-in bearded performer (or a stock Santa-like face) on Tavus 4.5 Pro, Anam Cara-4.1 and HeyGen LiveAvatar. Wire all three into one LiveKit agent with a vendor flag.
2. **Day 2–4.** Run 300+ scripted turns per vendor from US-East and US-West, on iPhone Safari over LTE and Wi-Fi and on a laptop with Chrome. Measure:
   - glass-to-glass turn latency (p50 and p95);
   - avatar-stage latency (LiveKit avatar metrics);
   - interruption stop time;
   - A/V offset;
   - time to first frame;
   - error and reconnect rate.
3. **Day 4–5.** Show blinded clips to about 20 parents, ideally with kids, and rate realism, warmth, beard quality, listening liveliness and creepiness. Kids are the toughest critics of the uncanny valley.
4. **Day 5.** Ask vendors for written answers on: reserved concurrency for Dec 20–26, data retention of our Santa audio, use of the data for training (we opt out), child-audience acceptability, and portrait support.
5. **Decision on Nov 14.** Primary = the best realism among vendors that meet the p95 avatar-stage limit of 250 ms and the 200 ms interruption limit. Secondary = the next best.
6. **Nov 9–12.** Real Santa shoot. **Nov 15–20.** Train production faces on two or three vendors.
   - Tavus 4.5 tuning takes a few hours.
   - HeyGen and Anam turnaround: *U*. Allow for training failures (Tavus faces cannot be re-tuned after a failure: a new face is required).
7. **Nov 20 – Dec 1.** Integration, load test, safety red-team. **Code freeze Dec 3.**

---

## 8. Child, policy and privacy considerations specific to the avatar layer

- **Data minimization.** With our agent in the middle, vendors receive only Santa's synthetic audio and return Santa's video. **Do not enable vendor perception features** (Tavus Raven, D-ID's camera layer) that would send the child's camera feed to a vendor. If the child's camera is on at all, it should be optional and processed only by us, or not at all.
- **COPPA.** The amended rule is fully in effect (compliance date 2026-04-22). It bars using children's data to train AI as a condition of service. Get each vendor to agree in writing that no child data reaches it; if any does, it must not be retained or used for training.
- **Vendor terms.** Tavus's ToS make *us* responsible for use by minors. Its AUP has zero tolerance for content that exploits or harms minors, which our use does not do, but **get written confirmation from each vendor that a child-directed, parent-consented product is permitted** *(U for all vendors)*.
- **Disclosure.** Tell parents clearly that Santa is an AI character played by a digital likeness. Some jurisdictions now require synthetic-performer disclosure in *advertising* (NY from 2026-06-09; CA SB 1050 from 2027-01-01, *U*). Apply the same disclosure to marketing videos made with the avatar.
- **Vendor-side moderation.** Some vendors filter content in their own pipeline. Confirm that Santa saying words like "naughty", "chimney" or "sneak" does not trigger vendor-side filters that would mute the avatar (*U*).

---

## 9. Risks and mitigations

| # | Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|---|
| 1 | Vendor capacity collapses on Dec 24 (GPU shortage, an outage) | Med | Critical | Reserved concurrency in writing; three vendors behind LiveKit; booked time slots; Elf waiting room |
| 2 | The beard renders badly (smearing, mouth hidden, flicker) | Med | High | Test on Day 1; trim the mustache; a real beard, not a costume one; try image and video training paths |
| 3 | Real turn latency is far above vendor claims (e.g., Tavus's 5 s time to first frame suggests heavier infrastructure) | Med | High | Bake-off with glass-to-glass measurement; backchannel cues; pick the vendor on measured p95 |
| 4 | Interruption feels sluggish because the vendor buffers too much audio | Med | Med | Clause-level TTS chunks; measure the clear_buffer → stop time; choose the vendor on it |
| 5 | Vendor ships a model change in December (behaviour or quality drift) | Med | High | Pin model versions (e.g., Phoenix-4.5 Pro face IDs); contractual change freeze Dec 1–Jan 2 |
| 6 | Vendor discontinues a product (Decart retired its avatar models 2026-05) | Low-Med | High | Multi-vendor abstraction; keep the master Santa footage under our own ownership |
| 7 | Echo: Santa's voice from the phone speaker triggers barge-in | High (on phones) | Med | Browser AEC; a minimum barge-in duration; tests on device |
| 8 | A vendor refuses the Santa likeness or the child audience under its policy | Low-Med | High | Ask in writing this week; synthetic-Santa backup (with consent rules for synthetic subjects to be confirmed) |
| 9 | Actor consent or contract gaps (AB 2602, NY, SAG-AFTRA, NO FAKES) | Low | High | Counsel-reviewed replica licence; performer represented; specific uses |
| 10 | Mobile autoplay or Low Power Mode breaks the first impression | Med | Med | A "Tap to call Santa" gesture; QA matrix of iOS 17–19 and Android 13–16 |
| 11 | Santa looks different across vendors | Med | Low-Med | Same master footage; pin each family to one vendor |
| 12 | Uncanny valley scares younger kids (3–4) | Low-Med | High | Parent and kid panel testing; warm lighting; mid-shot framing (not an extreme close-up); a softer emotion palette |

---

## 10. Open questions / unverified items

- Tavus enterprise concurrency ceiling and regions for Dec 2026; whether Phoenix-4.5 Pro supports portrait output; stop latency on `clear_buffer`; acceptance of a Santa-costumed actor (**U**).
- Anam's published concurrency limits, pricing tiers, custom-avatar input requirements and child-audience policy (**U**).
- HeyGen LiveAvatar's transport and real measured turn latency (vendor says under 300 ms to first frame; Anam measured a 3.66 s median connect) (**U**).
- Decart Lucy 2.6 latency and fps under load, and whether any successor audio-driven avatar model is planned (**U**). None was found in the changelog through Oct 2026.
- Synthetic-subject rules (an AI-generated Santa face) at Tavus, Anam and HeyGen (**U**).
- Per-component licences for NVIDIA Audio2Face (SDK under MIT vs models under the NVIDIA Open Model License); not relevant to launch (**U**).

---

## 11. Sources

**Decart**
- Decart changelog (retirements, Lucy 2.6, fast mode): https://docs.platform.decart.ai/changelog
- Decart models table: https://docs.platform.decart.ai/getting-started/models
- Lucy 2.6 realtime docs: https://docs.platform.decart.ai/models/realtime/lucy-2.6.md
- Decart pricing: https://docs.platform.decart.ai/getting-started/pricing.md
- Decart Lip Sync API launch (Oct 2025): https://x.com/DecartAI/status/1981078296084488293
- Decart + ElevenLabs living characters: https://decart.ai/publications/building-living-characters-with-decart-and-elevenlabs
- Lucy 2.6 coverage: https://alphasignal.ai/news/decart-s-lucy-2-6-stops-ai-video-faces-and-clothes-from-drifting ; https://github.com/DecartAI/decart-ios/pull/53
- Lucy 2 (Forbes, Jan 2026): https://www.forbes.com/sites/charliefink/2026/01/27/decarts-new-lucy-2-generative-ai-video-model-pushes-generative-video-into-real-time/

**Tavus**
- Research (Phoenix-4.5 134 ms, Sparrow-2, Raven-1): https://www.tavus.io/research
- Phoenix-4 launch coverage: https://www.marktechpost.com/2026/02/18/tavus-launches-phoenix-4-a-gaussian-diffusion-model-bringing-real-time-emotional-intelligence-and-sub-600ms-latency-to-generative-video-ai/
- CVI overview: https://docs.tavus.io/sections/conversational-video-interface/overview
- Training paths / requirements: https://docs.tavus.io/sections/faces/which-training-path.md
- Phoenix-4.5 preview & tuning status: https://docs.tavus.io/sections/faces/phoenix-45-preview-and-status.md
- Pipeline modes: https://docs.tavus.io/sections/conversational-video-interface/quickstart/pipeline-modes.md
- LiveKit integration: https://docs.tavus.io/sections/integrations/livekit.md
- Background customization (not on 4.5): https://docs.tavus.io/sections/conversational-video-interface/conversation/customizations/background-customizations.md
- Acceptable use policy: https://tavus.io/acceptable-use-policy ; Terms: https://tavus.io/terms-of-service
- Pricing / caps (third-party trackers, unverified): https://www.tavus.io/pricing ; https://www.usagepricing.com/blueprint/activity/tavus-2026-10-24-conversation-duration-caps ; https://pricingsaas.com/companies/tavus/diffs/2026W40

**Anam**
- Cross-vendor connection benchmark (Oct 16, 2026; conflict of interest disclosed): https://anam.ai/tools/latency-benchmark
- Cara-4.1 Express: https://anam.ai/blog/cara-4-1-express ; https://anam.ai/cara-4-1
- Cara-4: https://anam.ai/blog/meet-our-most-expressive-model-yet-cara-4
- API overview: https://anam.ai/api
- Custom TTS / audio passthrough & interruptPersona: https://docs.anam.ai/examples/custom-tts

**HeyGen LiveAvatar**
- Product/pricing/concurrency: https://www.liveavatar.com/
- FAQ: https://help.heygen.com/en/articles/12758866-liveavatar-faq ; https://help.heygen.com/en/articles/12758516-introducing-liveavatar
- HeyGen's own platform roundup (Oct 29, 2026; vendor-authored): https://www.heygen.com/blog/best-real-time-ai-avatar-platforms
- Moderation policy: https://www.heygen.com/moderation-policy

**Other vendors**
- LiveKit avatar plugin list & architecture: https://docs.livekit.io/agents/models/avatar/
- LiveKit Simli plugin: https://docs.livekit.io/agents/models/avatar/plugins/simli/
- LiveKit Synthesia plugin: https://pypi.org/project/livekit-plugins-synthesia/1.8.8/
- LiveKit Tavus plugin: https://pypi.org/project/livekit-plugins-tavus/
- clear_buffer RPC (third-party Go port of LiveKit avatar protocol): https://pkg.go.dev/github.com/plexusone/omni-livekit/avatar
- SpatialReal plugin (pause/resume interruption pattern): https://pypi.org/project/livekit-plugins-spatialreal/1.8.3/
- Runway Characters: https://runway.com/product/characters ; https://runwayml.com/news/building-runway-characters
- D-ID V4 launch: https://www.prnewswire.com/il/news-releases/d-id-launches-v4-expressive-visual-agents-for-real-time-llm-connected-interaction-at-enterprise-scale-302714463.html ; https://docs.d-id.com/docs/agents-sdk-overview
- LemonSlice 2.1 Flash: https://lemonslice.com/blog/lemonslice-flash
- Hedra Live Avatars: https://mpost.io/hedra-launches-live-avatars-with-livekit-delivering-real-time-ai-characters-at-ultra-low-latency-and-cost/ ; https://gamiways.memoways.com/en/platform/hedra/
- bitHuman: https://pypi.org/project/bithuman/1.7.7/ ; https://adtmag.com/articles/2025/08/13/bithuman-launches-cpu-based-avatar-sdk.aspx
- Protoface plugin launch: https://natlawreview.com/press-releases/protoface-launches-livekit-plugin-real-time-avatars
- Keyframe Labs + LiveKit: https://www.tipranks.com/news/private-companies/livekit-showcases-multi-agent-real-time-avatar-capabilities-with-keyframe-labs
- Model comparison (Aug 2026, third party): https://meetcody.ai/blog/real-time-ai-avatar-models-compared/
- Simli review: https://aitwin.me/blog/simli-avatars-review
- NVIDIA Audio2Face open source: https://www.tweaktown.com/news/107951/nvidias-ai-powered-audio2face-animation-model-is-now-open-source-for-all-developers/index.html ; https://huggingface.co/nvidia/Audio2Face-3D-v3.0
- UE 5.7 MetaHuman lip-sync issues: https://forum.convai.com/t/severe-lip-sync-and-latency-issues-in-ue-5-7/5992 ; https://forums.unrealengine.com/t/metahuman-xada-for-custom-audio-streams-guidance-needed/2675424

**Legal / policy**
- COPPA amended rule (Federal Register): https://www.federalregister.gov/documents/2025/04/22/2025-05904/childrens-online-privacy-protection-rule
- COPPA compliance date reminder: https://bbbprograms.org/media/insights/blog/coppa-amended
- California AB 2602 / AB 1836 guide: https://aiunderstanding.org/learn/california-digital-replica-laws-performers
- New York AI image laws (Skadden, Jan 2026): https://www.skadden.com/insights/publications/2026/01/two-newly-enacted-new-york-laws-will-regulate
- NO FAKES Act 2026: https://www.govtrack.us/congress/bills/119/s4591 ; https://www.hklaw.com/en/insights/publications/2026/06/senate-judiciary-committee-advances-legislation-to-protect-name
- Digital replicas & consent overview: https://fasthofflawfirm.com/blog/digital-replicas-synthetic-performers
