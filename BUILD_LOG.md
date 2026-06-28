# SwingCheck Build Log

> One line per change. This is the raw material for college essays, progress tracking, and accountability.  
> Started: June 2026 (backfilled from project history)

---

## June 2026
- Jun 1: Back from finals. Rebuilt 18-month plan (5 chapters, month-by-month timeline, metrics targets)
- Jun 1: Created BUILD_LOG.md and backfilled full project history
- Jun 1: Added analytics pipeline — page_load, analyze_start, analyze_complete events tracked via Apps Script to Google Sheets
- Jun 1: Anonymous session UUID (localStorage) for unique user counting
- Jun 1: Weekly digest script — auto-emails page loads, unique sessions, completion rate, avg score, club breakdown
- Jun 1: Debugged analytics delivery — fetch and Image beacon both fail with Apps Script redirects. Form POST to hidden iframe works (same pattern as waitlist). Shipped working version.
- Jun 1: Switched from Apps Script analytics to Google Analytics (G-REZZ4NT4ZQ). Removed all custom Apps Script tracking code. GA tracks page views automatically + custom events for analyze_start and analyze_complete.
- Jun 1: Built welcome email sequence — 3 automated emails (welcome on signup, tips after 2 days, nudge after 5 days). Daily trigger via Apps Script checks signup dates and sends on schedule.
- Jun 1: Created 2 social media posters — signup CTA (dark green) and how-it-works (warm sand, 3 steps).
- Jun 10: Upgraded Gemini model chain — gemini-3.5-flash (newest, May 2026) → gemini-3-flash → gemini-2.5-flash. Two generations newer than previous chain.
- Jun 10: Rebuilt AI coaching prompt — 28 swing observations (up from 6), real golf terminology (over the top, early extension, casting, chicken wing, coil, compression), club-specific coaching context, skill-level awareness, ball flight connections, and feel cues. Significantly more detailed and golf-literate coaching output.
- Jun 10: Added 9 more swing observations (37 total): reverse pivot detection, trail knee flex maintenance, address posture quality (hip hinge + knee flex), downswing transition sequencing (hips vs shoulders). Added DTL view limitations to prompt so AI doesn't hallucinate about club face, grip, or ball flight.
- Jun 10: Created 2 social media posters — "What's your swing score?" (score tiers) and "Looking to improve your swing?" (value prop).
- Jun 23: Fixed AI analysis failing on "high demand" — model fallback loop only advanced on 404/429, so a 503 (model overloaded) killed the whole analysis. Now falls through to the next model on 500/502/503, and on per-model timeout, instead of giving up. Clearer error messages when the full chain is genuinely busy.
- Jun 23: Fixed clearTimeout scoping bug — timeout was const-scoped to the try block, so a network error in the catch threw a ReferenceError that masked the real error.
- Jun 23: Fixed truncated AI coaching (cut off mid-sentence) — Gemini 3.x Flash thinks by default and on generateContent those thinking tokens share the maxOutputTokens budget, so thinking ate the 1200-token cap and the answer hit MAX_TOKENS. Now sets per-model thinking config (3.x: thinkingLevel LOW; 2.5: thinkingBudget 0) and raised maxOutputTokens to 4096.
- Jun 23: Hardened AI response parser — strips "thought" parts so reasoning can't leak into displayed coaching, and surfaces empty/truncated responses with a clear "Tap Retry" message instead of showing half an answer.
- Jun 23: Verified swing start/end detection already trims the video before analysis (detectSwingBoundaries → extractFrames window → AI gets only the 4 key frames). Found the real waste: MediaPipe ran over the whole video twice (boundary scan, then extraction). Now the boundary scan caches its full landmarks and extractFrames reuses them for ≤30fps sources — eliminates the second pose pass. Higher-fps clips still get a dedicated denser pass; auto-falls-back to fresh extraction if reuse yields too few frames.
- Jun 23: Added testable swing-detection summary — console "[swing]" group (window, % trimmed, frames, reused vs fresh) plus window.__swingDebug for inspection. Hoisted a per-frame canvas the boundary scan was recreating each iteration.
- Jun 23: Renamed metrics.xFactor → metrics.spineTilt everywhere (field, scoring, tips, pro data, Compare tab, stat-bar IDs, AND the AI payload/prompt). The value was always spine tilt; the stale "xFactor" label was telling the AI it was shoulder-hip separation, a different metric. Removed the now-redundant pro-data "spine" field.
- Jun 23: Fixed dead model fallback — chain had "gemini-3-flash" which 404s; correct API name is "gemini-3-flash-preview". Chain is now 3.5-flash → 3-flash-preview → 2.5-flash, three valid independent quotas.
- Jun 23: Made the analysis fps cap an explicit constant (MAX_EXTRACT_FPS = 60). Source video fps detection (e.g. 240fps slow-mo) is unchanged; we just don't analyze more than 60fps. Set willReadFrequently on the replay frame-cache canvas to clear the getImageData perf warning.
- Jun 23: Built swing-history storage layer — saves a compact record per analyzed swing (date, club, angle, score, core metrics, flagged faults) to localStorage, capped per golfer. Added summarizeSwingHistory() — pure-JS fixed-size trend roll-up (score trend, spine tilt/shoulder/hip rising-or-steady, recurring faults) for the same club. Foundation for personalized coaching; not yet wired into the AI prompt.
- Jun 23: Added on-device golfer profiles so one device can serve many people (e.g. coaching at the range). "Golfer" selector on the setup screen, per-golfer history (tagged with golferId, trimmed per golfer), a "Guest (don't save)" mode, addGolfer/removeGolfer, and a default "Me" profile so solo users never see complexity. No accounts, no backend — all localStorage. Exposed window.swingCheckApp for console testing.
- Jun 23: Discovered the AI never received raw frames — it reads plain-English observations our JS computes from the key frames. The old "trajectory frame" sampling was dead code (built, never wired into the prompt). Removed it.
- Jun 23: Added swing-plane analysis (DTL only). MediaPipe doesn't track the club, so this reads plane from the hands and lead arm — a proxy, not true shaft plane. Phase-anchored checkpoints (takeaway ~30% to top, lead arm at top, transition ~18% to impact) produce observations (steep/inside takeaway, laid-off/upright top, over-the-top vs into-the-slot transition) fed to the coach. Thresholds are fractions of shoulder width (camera-distance-independent) and live in this.PLANE for calibration; over-the-top uses drop-vs-horizontal ratios (orientation-independent). DTL-only, silent on low confidence/borderline reads. Coaching is upfront that confirming shaft plane needs the club in view.
- Jun 23: Fixed address detection collapsing onto the top. Velocity alone couldn't tell address from the top (both are quiet — the club pauses at the top), so address kept landing near the top. Now uses hand height (hands are DOWN at address, UP at top), and snaps to the last genuinely-still frame just before the takeaway begins (hands clearly rising off the ball). Added a console warning when address/top still collapse (clip starts mid-swing). This also fixes shoulder turn, the takeaway read, and plane analysis, which were comparing a frame against itself when the two collapsed.
- Jun 23: Fixed fps detection mislabeling real-speed clips as 240fps slow-mo. It divided decoded-frame-count by duration, but the seek-driven scan decodes extra frames, inflating the count. Now measures real frame timing via requestVideoFrameCallback (median/low-percentile gap, skipping decoder-startup frames) as the primary method; decoded-count is a fallback. The bogus 240 had been forcing a redundant 2nd pose pass and pushing detection toward its flat-velocity worst case.
- Jun 23: Rebuilt key-frame detection around hand-height STRUCTURE instead of velocity. Velocity is unreliable on slow-mo/low-fps (flat curve → ambiguous peak → swing compressed into a sliver). Now anchors on the hand path: takeaway = hands rise off the ball, top = hands highest, impact = hands lowest after top, all from the smoothed wristY curve. Velocity is median-filtered (kills single-frame landmark spikes) and used only as a coarse aid. Removed the velocity peak anchor entirely.
- Jun 23: Hardened frame delivery (cause of intermittent garbage/"messed up skeleton" runs). safeSeek now waits for the decoded frame to actually be painted (requestVideoFrameCallback) before capture, so it stops grabbing the previous frame; and the boundary scan now gates frames into the detection curves on overall pose quality.
- Jun 23: Fixed the address bug found by running the REAL MediaPipe model on a real swing (Python, frame-by-frame). Two bugs: (1) the scan's confidence gate required wristVis ≥ 0.3, but at address the wrists read low-confidence even though their position is stable — so every address frame was being thrown out. Changed to gate on overall pose quality. (2) Address now uses sustained STILLNESS (frame-to-frame hand speed below a peak-relative threshold) to find the last quiet frame before the takeaway, tolerant of a gentle waggle. On the test swing this moved address from a wrong 1.46s to the true ~1.21s setup.
- Jun 23: Fixed key-position photo/skeleton mismatch (skeleton showed the right position but the photo showed a different frame). Cause: skeleton came from scan-time landmarks, photo from a separate re-seek that could land a frame off (seek imprecision + a race with background frame caching). Now recomputes pose on the exact displayed frame so the skeleton can never disagree with the photo.
- Jun 23: Removed dead code — getInterpolatedProSkeleton (~67 lines) and getWristPosition (~8 lines), both orphaned leftovers from old ghost-mode work, verified unreferenced.
- Jun 23: Made fps detection use real frame timing (requestVideoFrameCallback) as the primary method; the old decoded-frame-count/duration method was demoted to a fallback because the seek-driven scan inflated the count and mislabeled real-speed clips as 240fps slow-mo.
- Jun 23: Replaced seek-based frame capture with deterministic play-through capture. Seeking isn't frame-accurate, so the same clip could land on slightly different frames run-to-run (the residual inconsistency). Now plays the clip once at 0.4× (so pose keeps up, no skipped frames) and grabs each decoded frame via requestVideoFrameCallback, gated to ~30fps by true media time. Self-correcting: if the play-through under-captures (e.g. file:// or blocked autoplay), it automatically falls back to the seek scan instead of collapsing. Fixed a hard collapse (all positions on one frame, score 0) caused by an under-capturing play-through with no fallback.
- Jun 23: Fixed systematic impact-late bias. detectKeyFrames searched ±2 frames for the highest-quality pose; at impact the true frame is the most motion-blurred (lowest quality), so the search drifted impact forward onto a clearer post-contact frame. Impact now takes the nearest frame in time (no quality drift); address/top/finish keep the quality search since the body is near-still there. Result: impact centered on true contact (±1 frame at 30fps) instead of biased late.
- Jun 23: Wired swing history into the AI coaching prompt (closing a long-standing open loop). summarizeSwingHistory() was computed but never sent to Gemini, so coaching was generic across sessions. The prompt now includes a fixed-size rollup (recent scores, metric trends, recurring faults) plus a "returning student" instruction to name trends, prioritize recurring faults, and credit genuine improvement. Computed in JS (no extra API call); omitted for guests and golfers with no baseline.

## April 2026

### Week of Apr 14–17 (Launch Week)
- Apr 17: **BETA LAUNCH DAY** — deployed to GitHub Pages, sent waitlist email, posted on TikTok + Instagram
- Apr 17: Created launch email (HTML + Apps Script sender) and Instagram launch poster (light editorial style)
- Apr 17: Created TikTok/Instagram bios with app link
- Apr 17: Updated landing page — linked "Try It Now" buttons to app, replaced X-Factor → Spine Tilt on landing page, updated copyright to 2026
- Apr 16: Fixed "More Options" dropdown — CSS class mismatch (.visible vs .open)
- Apr 16: Changed hand path trail colors from green to gold (lead) and purple (trail) to differentiate from skeleton
- Apr 16: Replaced X-Factor with Spine Tilt everywhere — calculation (atan2 hip-to-shoulder), scoring, tips, compare tab, AI prompt, pro data values
- Apr 16: Code quality sweep — 3 Outfit→DM Sans font fixes, touch-action on score reveal CTA, removed duplicate CSS property, cleaned triple blank lines
- Apr 16: Full code audit — checked for alert(), debugger, localhost refs, empty catches, orphaned IDs, XSS risks, iOS Safari quirks, reduced-motion support
- Apr 16: Stripped 17KB dead skeleton code — skeletonFaceOn, skeletonDTL, initGhostMode(), drawGhostSkeleton(), mirrorSkeleton()
- Apr 16: Rewrote ghost mode render dispatch — removed orphaned drawGhostSkeleton fallback call
- Apr 16: Added placeholder image fields for Nelly Korda (clean "Photo coming soon" fallback)
- Apr 16: Removed Scottie's duplicate imagesDTL block (saved 143KB)
- Apr 16: Upgraded Tiger Woods photos — replaced all 4 DTL images with higher-res versions (381×492 → 797×1067)
- Apr 16: Fixed Compare tab rendering — cover fit for Rory/Scottie, proper scaling for Tiger
- Apr 16: Fixed Compare tab labels — CSS selector mismatch (.frame-label → .comparison-frame-label), moved labels to top-left pills, removed black bar overlay
- Apr 16: Changed canvas wrapper background from #000 to warm off-white
- Apr 16: Fixed pro photos root cause — all 16 images mislabeled as image/png but were WebP format. Fixed MIME types
- Apr 16: Built Phase 2 score reveal — full-screen Apple Watch-style overlay with SVG ring fill, count-up animation, skeleton draw-in, particle burst, tier labels (Tour Quality/Great/Building/Developing), "See Full Analysis" CTA
- Apr 16: Score reveal respects prefers-reduced-motion, replays cleanly on re-analysis

### Week of Apr 7–13 (Pre-launch Polish)
- Apr 13: Phase 1 frontend redesign complete — new color system (#006747 green, #D4A843 gold), DM Sans typography, screen transitions, micro-interactions
- Apr 13: Face-On camera angle disabled with "SOON" badge
- Apr 13: Fixed crop/boundary detection issues
- Apr 13: Added feedback submission system
- Apr 13: SC monogram logo with radiating gold arcs finalized
- Apr 13: Fixed icon sizing throughout app
- Apr 13: Fixed video reset bug on re-analysis
- Apr 13: Fixed pro canvas scaling on Compare tab

### Week of Apr 1–6 (Landing Page + Email)
- Apr 8: Built landing page — hero, features (stacked cards), how it works, ghost mode section, builder story, roadmap, FAQ, waitlist form
- Apr 8: Waitlist connected to Google Sheets via Apps Script
- Apr 8: Email system set up — HTML template + Apps Script sender from Gmail
- Apr 8: Added background visuals to landing page — swing arcs, pose landmarks, angle measurements, golf quotes
- Apr 8: Added "Built by a high school golfer" section with builder tags
- Apr 8: Added public roadmap (Now / Next Up / Future)

## March 2026

### Early Development
- Built core swing analysis engine — MediaPipe pose detection (33 landmarks), 4-phase detection (address, top, impact, finish)
- Implemented confidence-filtered frame extraction
- Auto FPS detection for uploaded videos
- XOR-encrypted Gemini API key with model fallback chain (flash-lite → flash → 3-flash-preview)
- AI coaching integration — sends landmark coordinates to Gemini, returns personalized feedback
- Ghost Mode — overlay Tiger Woods skeleton on user's swing (PRO_SKELETON_DATA with real MediaPipe frames)
- Frame-by-frame replay with skeleton overlay
- Drawing tools — lines, circles, freehand annotation on replay canvas
- Reference lines toggle
- Hand path visualization
- Handedness selector (left/right)
- Club type selector (driver, wood, hybrid, iron, wedge)
- Scoring system (0-100) with weighted metrics
- Stats display — shoulder turn, hip turn, tempo, swing plane
- Compare tab — side-by-side with pro golfers (Tiger, Rory, Scottie, Nelly)
- Tips tab — context-aware coaching based on metrics
- Drills tab — specific practice drills based on detected issues
- Speed controls for replay
- Impact offset adjustment
- Auto crop toggle
- Safe video seeking (prevents iOS Safari race conditions)
- Error overlay for graceful failure states
- Single HTML file architecture (~670KB self-contained)

---

## How to update this log

After every work session, add a line:
```
- [Date]: [What you shipped]
```

Keep it factual. No fluff. Future you will thank present you.
