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
