# Panel Slot Finder

A reconciliation engine for interview panel scheduling. I wanted to flex my skill set and work through a case study end to end, so I built the one piece of the scheduling problem worth building rather than a thin slice of all of it.

**Live tool:** https://itworkssly.github.io/panel_slot_finder/

## What it does

Interview coordinators spend their day reconciling four moving inputs by hand: who is candidate-ready, who is trained to interview for which competency, who is actually free, and what the panel shape needs to be. Panel Slot Finder takes that reconciliation and does it in one pass. It reads availability and interviewer state, matches them against the panel a candidate needs, and returns a confirmed slate with coverage called out honestly.

The wedge is not scheduling. Scheduling is table stakes; most coordination tools do it. The wedge is **train-or-override coverage plus pool-depth analytics**. The tool never fakes coverage. When a competency pool is too thin to staff a panel cleanly, it routes the gap to a train-or-override decision instead of quietly filling the slot with someone untrained. That distinction is the whole point.

## Architecture and scope

It is a standalone reconciliation engine, written as a single-file HTML/CSS/vanilla-JS app. The parse and scheduling primitives are ATS-agnostic. I optimized the reference implementation for Ashby, but the primitives apply to Greenhouse, Lever, or any ATS with the same underlying objects (reqs, candidates, interviewers, competencies).

## What is real and what is simulated

I want this clear up front.

- **Real:** the parse logic and the scheduling math. The aggregations that drive the dashboard are computed live from the seeded sprint, not typed in.
- **Real:** live parse mode makes an actual Claude API call against pasted text, using your own API key.
- **Baked:** demo mode runs preloaded sample scenarios with no key required, so you can see how the tool behaves without setting anything up.
- **Simulated:** the calendar pull and the ATS write-back. The write-back shows the exact payload it would send to the ATS. There is no live Ashby integration behind it, and I am not implying one.

## Running it

It is a single HTML file. Open the live link above, or clone and open `index.html` in a browser.

- **Demo mode** needs nothing. Click through the baseline and the two problem scenarios.
- **Live parse mode** needs your own Anthropic API key, entered in the tool. The key stays in your browser and drives only the parse call.

## The dashboard

Five tiles, all derived from the seeded sample sprint (25 active reqs, roughly 75 panels, roughly 225 interviews over a two-week sprint):

1. **Coordinator hours saved.** Manual touches per panel versus touches with the tool, across the sprint. The baseline carries an illustrative label.
2. **Coverage.** Percent of interviews staffed, with gaps routed to train-or-override and never filled untrained.
3. **Interviewer load by pool.** Interview-hours per trained person per week, by competency pool. Thin pools flag against a burnout line. Strain is always shown as a pool, never a named person.
4. **Reschedule pattern.** Where slips cluster, which is on the thin pools.
5. **Scheduling time.** Labeled a staffing signal, not a tool metric, because it mostly measures calendar density.

## Skills this exercised

- Problem decomposition: mapping the full candidate-ready-to-confirmed workflow and picking the one piece worth building
- Scheduling and constraint logic in vanilla JS
- Deterministic test data via a seeded PRNG so every render is identical
- Claude API integration for the live parse path
- Aggregation and metric design driven from a single source sprint
- Honest metric framing: cutting weak metrics, labeling illustrative versus computed
- Single-file front-end build, dark mode, accessible focus states
- Pre-ship validation: syntax checks, inline logic tests, stale-phrase grep
- ATS data modeling against Ashby objects with portable primitives
- Scoping discipline: deciding what to leave out and saying so
- Stakeholder framing for both an operator audience and an AI-judgment audience
- API-key handling kept client-side

## What is deliberately left out

Named here as foresight, not omission:

- Full timezone support pulled from each interviewer's calendar. The prototype converts candidate hours only.
- A candidate-facing view that flags conflicts before lock.
- Drag-to-reorder the panel.
- Live ATS write-back against a real Ashby sandbox. Currently simulated.
