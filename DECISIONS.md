# SwingCheck — Engineering Decisions

> A running record of the **decisions** behind SwingCheck — especially how I evaluate new
> AI tooling as it ships and decide what to adopt, what to build myself, and what to skip.
>
> This is **not** a changelog. The day-to-day "what I shipped" lives in `BUILD_LOG.md`.
> This file is for the smaller number of moments where I had to *make a call*: a new model
> dropped, a framework got hyped, an architecture needed rethinking. The point isn't to
> show I used the newest thing — it's to show I can reason about the newest thing.

---

## How to use this log

- **Add an entry when you make a real decision or evaluate a new tool** — not for every code change.
  Aim for roughly one entry every week or two. A few sentences of *reasoning* beats a paragraph of summary.
- **Write it the day you decide, while the reasoning is fresh.** Backfilled reasoning reads like backfilled reasoning.
- **A strong entry shows a tradeoff you weighed and rejected**, not just the thing you picked. The rejected
  option is where the judgment shows.
- **Note what would make you revisit the decision.** Tech depreciates; saying so proves you know it.

### Entry template

```
## YYYY-MM-DD — <short title of the decision>

**What prompted this:** <the problem, or the new tool/model that dropped>

**Options I weighed:**
- <option A>
- <option B (often the hyped/obvious one)>

**What I found:** <the actual reasoning — the part that matters>

**Decision:** <what I did, and the one-line why>

**Would revisit if:** <what change would flip this>
```

---

## 2026-06-23 — Deterministic frame capture: play through, don't seek

**What prompted this:** After the address fix, the same clip still gave slightly different key positions
run-to-run. Traced to frame capture: the scan seeked to timestamps, and browser seeking isn't
frame-accurate, so the same request could land on a neighboring frame each run.

**Options I weighed:**
- Keep seeking, accept the wobble (cheapest; positions are structure-based so it's usually ±1 frame).
- Play the clip through once and capture each decoded frame in order (deterministic by construction).
- WebCodecs / VideoDecoder (gold standard, decodes every frame with no timing dependence) — biggest rewrite.

**What I found / decided:** Went with play-through capture. The catch is that pose inference (~40ms) is
slower than a frame interval, so playing at full speed would skip frames non-deterministically — so it
plays at 0.4× (pose keeps up) and gates to ~30fps by true media time (a 60fps clip doesn't double the
work). Crucially, play-through is fragile on some environments (file:// URLs, blocked autoplay), where it
under-captures — so it is SELF-CORRECTING: if it captures too few frames it falls back to the seek scan
rather than proceeding with garbage. (A version without that fallback caused a hard collapse — all
positions on one frame, score 0 — on a file:// test.)

**Would revisit if:** wobble persists even with play-through → move to WebCodecs, which removes the
real-time-playback dependency entirely.

---

## 2026-06-23 — Impact takes the nearest frame, not the cleanest

**What prompted this:** Impact landed 1–2 frames late, consistently.

**What I found:** The key-frame picker searched ±2 frames for the highest-quality pose. That's fine for
slow positions (address/top/finish), but impact is the most motion-blurred frame in the swing (fastest
point), so the cleanest nearby frame is always just AFTER contact — the search systematically pushed
impact late.

**Decision:** Impact takes the nearest frame in time (accept a slightly blurry skeleton there; the moment
matters more than the picture). Slow positions keep the quality search. This centers impact on true
contact (±1 frame at 30fps — the frame-rate limit). Tighter than that needs sub-frame work (audio
transient on clips that have audio, or hand-curve interpolation) — deferred until precision demands it.

**Would revisit if:** impact precision needs to beat ±1 frame → add audio-transient anchoring (only works
on clips that still have their audio track; re-encoded/edited clips often don't).

---

## 2026-06-23 — Coaching remembers: wire swing history into the prompt

**What prompted this:** A profile had 33 saved swings, but the coaching read the same generic idea every
time — correct about the current swing, blind to the arc. The history was being SAVED but never SENT to
the model.

**Options I weighed:**
- Send the raw history dump (simple, but burns tokens and grows unbounded).
- Send a small fixed-size SUMMARY computed in JS (trends, recurring faults, recent scores).
- Jump straight to an "agent" that reasons over history (premature — the memory is still thin).

**Decision:** Fixed-size JS summary, injected as a "their recent history" section plus a "returning
student" instruction (name trends, prioritize recurring faults, credit real improvement, never invent
history). No extra API call; omitted for guests and no-baseline golfers. This is the architectural moment
the product crosses from single-swing ANALYZER to coach-that-REMEMBERS — the actual moat, since anyone can
call a vision model on one swing but the longitudinal memory is what's hard to copy. The agent layer comes
LATER, once the memory is rich and the coaching reads it; built in the other order the agent is a shell
around thin data.

**Caveat to watch:** trend quality depends on stored metrics being consistent across swings; history saved
before the detection fixes is noisier, so the recent window is the trustworthy part. If the model ever
cites a trend that isn't in the summary, tighten the instruction.

**Would revisit if:** history grows large enough that the recent-5 window misses meaningful longer-term
patterns → widen the summary window or add per-fault streak tracking.

---

## 2026-06-23 — Debug detection against the real model on a real swing (and what it found)

**What prompted this:** Address detection was landing late and inconsistently, and tuning in the dark
wasn't converging. Decided to run the actual MediaPipe model on a real clip, frame by frame, in Python —
deterministic ground truth, no browser.

**Options I weighed:**
- Keep adjusting thresholds against screenshots (guessing).
- Extract the real hand-height + confidence curve from a real swing and debug against it.

**What I found:** The data exposed two bugs guessing never would have. (1) A confidence gate I'd added
required wrist visibility ≥ 0.3, but at address MediaPipe reports low wrist *confidence* (hands together/
low/occluded) even though wrist *position* is rock-stable — so the gate was deleting every address frame,
and the detector didn't see the swing until it was already mid-takeaway. (2) Hand height alone can't place
address precisely because the hands barely move in position at the very start of the takeaway; frame-to-
frame *stillness* separates setup from takeaway cleanly.

**Decision:** Gate scan frames on overall pose *quality*, not wrist visibility. Detect address as the last
sustained-still frame (hand speed below a peak-relative threshold) before the takeaway. Adopt "run the real
model on real data" as the debugging method of first resort for detection issues, not last.

**Would revisit if:** a future pose model reports confidence differently, or stillness thresholds prove
fragile across very different swings (they're tunable and peak-relative to mitigate this).

---

## 2026-06-23 — Hand-height structure for key-frame detection, not velocity

**What prompted this:** On slow-mo / low-fps clips the detector compressed the whole swing into a fraction
of a second. Root cause: it used peak hand *velocity* to anchor the downswing, but in slow-mo every speed
is low, so the "peak" is ambiguous and everything keys off the wrong frame.

**Options I weighed:**
- Keep velocity anchoring and try to smooth the curve.
- Detect from the hand-height *structure* (down→up→down→up); the four positions are its turning points.
- Multi-signal fusion / template matching (more robust in theory, much more complexity).
- AI (Gemini) as the detector.

**What I found:** Hand height has a fixed shape regardless of playback speed, and position barely jitters
even with landmark smoothing off — far more robust than velocity. Fusion/templates were over-engineered for
the gain. AI-as-detector is costly, slow, and (being a vision model) *inconsistent* run-to-run — the exact
problem we're fighting; AI belongs as a verifier layer, not the detector.

**Decision:** Detect positions from the hand-height curve; median-filter velocity and use it only as a
coarse aid. Keep AI-as-verifier as a future layer, not the primary detector.

**Would revisit if:** club tracking becomes available (enables true velocity/plane), or face-on view needs
a different structural signal.

---

## 2026-06-23 — Reproducibility: smoothing off, recompute pose for display

**What prompted this:** Same clip gave different results each run, and key-position photos sometimes didn't
match their skeletons.

**What I found:** Two separate nondeterminism sources. (1) MediaPipe's landmark smoothing carries state
across the frame *sequence*, so seek-timing jitter changed results run-to-run — turning it off makes each
frame independent and reproducible. (2) The displayed photo was re-seeked separately from the scan-time
landmarks, so they could land on different frames (seek imprecision + a race with background caching).

**Decision:** Disable landmark smoothing (reproducibility over a little extra jitter, which our own
filtering handles). For display, recompute pose on the exact captured frame so the skeleton can never
disagree with the photo. The deeper cure for residual seek jitter — playback-based frame capture instead of
seeking — is noted but deferred until it's shown to be necessary.

**Would revisit if:** residual run-to-run wobble persists → switch the scan from seeking to playback-based
capture (play once, grab each decoded frame via requestVideoFrameCallback).

---

## 2026-06-23 — Coaching reads observations, not raw frames; "seeing" plane without a club

**What prompted this:** Wanted richer backswing/transition tips. Digging in revealed two things:
the AI never receives raw frames at all (it gets plain-English observations my JS computes from the
key frames), and the old "trajectory frame" sampling was dead code — built but never wired into the
prompt. Also, MediaPipe tracks the body but **not the club**, so true shaft plane isn't measurable.

**Options I weighed:**
- **Send more raw frames/coordinates to the model** and let it reason about plane.
- **Compute plane observations in JS** from intermediate frames and feed them as English.

**What I found:** LLMs are unreliable at geometric reasoning over raw x/y, and the prompt already
forbids mentioning coordinates — so dumping frames would add tokens and hallucination risk, not insight.
The leverage is *more and better observations*, computed where the geometry is reliable (JS). On plane
specifically: I can't measure shaft plane (no club), but the hand path and lead-arm angle from
down-the-line are a strong proxy for the plane faults a coach actually calls. Key robustness choices:
thresholds are **fractions of shoulder width** (camera-distance-independent, not raw frame fractions),
and the over-the-top vs in-the-slot read uses **vertical-drop-vs-horizontal-travel ratios** rather than
absolute frame directions (which depend on clip orientation).

**Decision:** Compute body-scaled plane observations (takeaway, lead-arm at top, transition) in JS from
phase-anchored intermediate frames; feed them as observations, not coordinates. DTL-only, silent on low
confidence or borderline reads, all thresholds tunable in `this.PLANE`. Removed the dead trajectory code.
The coaching is upfront that confirming true shaft plane needs the club in view.

**Would revisit if:** club tracking (a club-detection model) or the face-on camera arrives — either
unlocks true shaft-plane measurement, at which point these hand/arm proxies become a fallback, not the
primary read.

---



**What prompted this:** Built per-swing history for personalization, then realized the use case
breaks it: I run *many different people's* swings through one device at Back Nine. A single shared
history blends everyone together — "your spine tilt is trending up" across 5 different golfers is noise.

**Options I weighed:**
- **Accounts + backend** (Firebase/Supabase + sign-in) — proper multi-user, cross-device, but a big
  infrastructure jump, holds users' data on a server, and overkill for "one coach, many students."
- **On-device golfer profiles** — a small list of named profiles in localStorage; tag each swing with
  a `golferId`; a "Guest (don't save)" option for demos and one-offs.

**What I found:** The accounts path solves a problem I don't have yet (cross-device sync) at the cost of
real infrastructure and data-custody responsibility. The on-device version solves the *actual* problem
(histories must not blend) with zero backend, keeps all data in the browser, and is arguably a better
privacy story. History is trimmed *per golfer* so a busy coaching session doesn't evict other students.
It also defaults to a single "Me" profile, so a solo end-user never sees the complexity.

**Decision:** On-device profiles with a `golferId` on every record. Default single profile; Guest mode
skips saving. Defer accounts/backend until there's a real cross-device need (Chapter 4 or later).

**Would revisit if:** users genuinely need their history across devices, or premium cross-device coaching
becomes the product — at which point a backend with accounts earns its complexity.

---



**What prompted this:** Planning a personalized AI coach. The obvious "frontier" move is to adopt
Nous Research's **Hermes Agent** — a real, self-improving agent framework (Feb 2026, MIT-licensed,
runs on a $5 VPS, Telegram/Discord, persistent memory). It's the hyped option right now.

**Options I weighed:**
- **Adopt Hermes Agent** — full framework, VPS, messaging integration, per-user memory.
- **Build a client-side agent loop myself** — Gemini function-calling with my own tools
  (`getSwingHistory()`, `getKnowledgeSection()`, `compareToProPattern()`), running in the existing app, no backend.

**What I found:**
- The actual *value* of personalization (tracking each golfer's swing trends and grounding coaching in
  a knowledge base) lives in the data + prompt, not in the framework. The framework is a delivery layer.
- Betting a **November 2027** application narrative on a **May 2026** framework is risky — this space moves
  monthly. OpenClaw was the dominant agent framework and partially imploded over security
  (9 CVEs in 4 days, one rated 9.9; 135k+ exposed instances). Internet-facing agents holding user data
  are a real responsibility, not a footnote.
- Building the loop myself means *I'm* demonstrably the engineer — an interviewer can probe it and I'll
  shine, instead of "what did you build vs. what did the framework do?"

**Decision:** *Pending my final call.* Leaning toward the client-side loop first; revisit full Hermes
as a Chapter 3 stretch goal **if** in-app personalization proves it's actually valued by users.

**Would revisit if:** in-app personalization clearly lands and users want cross-device history +
a chat channel — at which point a server-backed agent earns its complexity.

---

## 2026-06 — Per-model thinking config for Gemini 3.x

**What prompted this:** Upgraded the coaching model to Gemini 3.x Flash. Coaching responses started
getting cut off mid-sentence.

**Options I weighed:**
- Just raise `maxOutputTokens` until it stops truncating.
- Understand *why* it truncates and configure for it.

**What I found:** Gemini 3.x Flash models reason ("think") by default, and on the `generateContent`
endpoint those thinking tokens share the same `maxOutputTokens` budget as the answer. My 1200 cap was
being eaten by thinking → `MAX_TOKENS` truncation. Also: 3.x controls reasoning with `thinkingLevel`,
while 2.5 uses `thinkingBudget` — and sending both to a 3.x model is an error, so the configs can't be identical.

**Decision:** Per-model generation config — `thinkingLevel: LOW` for 3.x, `thinkingBudget: 0` for 2.5,
and raised the cap to 4096 so the full answer fits with room for reasoning.

**Would revisit if:** a higher thinking level measurably improves coaching quality enough to justify the
extra latency/cost, or if Google changes the default reasoning behavior.

---

## 2026-06 — Resilient model fallback chain

**What prompted this:** Free-tier Gemini models intermittently return 503 (overloaded) at peak times,
and real error logs showed a single 503 was killing the entire analysis.

**Options I weighed:**
- Retry the same model on failure.
- Fall through a chain of independent models (separate quotas).

**What I found:** My fallback only advanced on 404/429, so a 503 broke the loop and surfaced a raw error.
The logs also caught a dead entry: `gemini-3-flash` was 404ing because the correct API name is
`gemini-3-flash-preview`. So I effectively had two working models, not three.

**Decision:** Fall through on 500/502/503 (and timeouts) too; fixed the model name. Chain is now
`gemini-3.5-flash → gemini-3-flash-preview → gemini-2.5-flash` — three independent quotas for resilience.

**Would revisit if:** models GA or deprecate (2.5-flash is aging) — this chain needs periodic maintenance,
which is itself a reason to keep this log.

---

## 2026-06 — One pose pass instead of two (swing-window reuse)

**What prompted this:** Analysis ran MediaPipe pose detection over the *entire* video to find the swing
boundaries, then ran pose *again* on the trimmed window — a double pass that scaled with total clip length.

**Options I weighed:**
- Leave it (simple, but wasteful).
- Cache the landmarks from the boundary scan and reuse them for the swing window.

**What I found:** The boundary scan already computed full landmarks for every frame — it just threw them
away, keeping only wrist position. For typical phone video (≤30fps) the second pass was pure waste.

**Decision:** Cache the scan's landmarks; reuse them for the window when source fps ≤ scan fps, with an
auto-fallback to a fresh pass if reuse yields too few frames. Eliminates the second pass for the common case.

**Would revisit if:** I add a detection method that needs higher temporal resolution than the 30fps scan provides.
