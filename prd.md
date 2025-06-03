## Product Requirements Document

**Feature:** Goal-Setter & Quick-Debrief Loop
**Product:** ForgeOn IQ-Tracker (Bowler + Batter editions)
**Author / DRI:** *Devansh (PM) – v 1.0*
**Date:** 3 June 2025

---

### 1  Purpose & Hard Truth

> Without a player-owned objective and a data-anchored reflection, every micro-over becomes a random quiz.
> Goal-Setter locks the athlete on *one* focus skill up-front; Quick-Debrief forces a 60-second accountability check at the end.
> Outcome: tighter learning loop, richer coaching context, measurable confidence delta.

---

### 2  Problem Statement

1. Current sessions lack **personal intent** → motivation drifts, sub-score gains plateau.
2. Coaches can’t tell *why* a player’s CQS/BIS spiked or tanked on a given day.
3. Adaptive engine tunes difficulty without human feedback → may over/under-adjust.

---

### 3  Goals & Non-Goals

| #  | Goal (Done when…)                                                  | Metric / KPI                                | Non-Goal                           |                     |                                                |
| -- | ------------------------------------------------------------------ | ------------------------------------------- | ---------------------------------- | ------------------- | ---------------------------------------------- |
| G1 | ≥ 95 % sessions include a focus skill & debrief answers            | `session_goals.count / sessions`            | Building a long reflective journal |                     |                                                |
| G2 | Reduce average **Confidence-Gap Score** (CGS) ±15 → ±10 in 4 weeks | \`avg                                       | CGS                                | \` week-4 vs week-0 | Guaranteeing performance uplift inside matches |
| G3 | Deliver difficulty tuning accuracy ±1 s w/in 3 sessions            | % sessions where player marks “About right” | Real-time coach chat               |                     |                                                |

---

### 4  Target Users & Personas

* **Bowler / Batter** – elite / academy; 8-min daily IQ sessions.
* **Coach** – needs context for prescribing on-ground drills.
* **Sports Psych** – tracks confidence trends vs match form.
* **System (adaptive engine)** – needs qualitative difficulty signal.

---

### 5  User Stories (critical subset)

| ID   | As a…  | I want…                                | Acceptance criteria (truncated)                |
| ---- | ------ | -------------------------------------- | ---------------------------------------------- |
| SE-1 | Player | 60 s Goal-Setter modal before training | Cannot skip • Saves `focus_skill` / confidence |
| SE-2 | Player | 60 s Quick-Debrief modal after session | Slider, difficulty vote, audio insight saved   |
| SE-3 | System | Compute CGS & difficulty rule          | p95 < 200 ms • Unit test passes                |
| SE-4 | Coach  | See Goal Fidelity (%) column           | Grid loads < 2 s • Sortable                    |
| SE-5 | Psych  | Monday CSV: confidence\_gain           | Cron email by 06:00 IST                        |

(Full backlog in Jira project **IQT-GS**.)

---

### 6  Functional Requirements

| Area    | Requirement                                                                                    |
| ------- | ---------------------------------------------------------------------------------------------- |
| **F-1** | Goal-Setter modal pops automatically after successful login & before Self-Eval.                |
| **F-2** | Focus-skill list loaded from YAML (hot-reload, versioned).                                     |
| **F-3** | Hard cap 180 s total interaction (countdown visible).                                          |
| **F-4** | Offline tablets: cache responses in IndexedDB → sync on reconnect.                             |
| **F-5** | Quick-Debrief captures audio (30 s max, auto-upload MP3).                                      |
| **F-6** | `/goal` and `/debrief` endpoints return 200 + session JWT refresh.                             |
| **F-7** | Adaptive engine: `session_difficulty = 'easy'` → timer -0.5 s next session; `'hard'` → +0.5 s. |
| **F-8** | Coach dashboard updates Goal Fidelity (%) real-time via WebSocket.                             |

---

### 7  Non-Functional Requirements

| NFR                                                                     | Target |
| ----------------------------------------------------------------------- | ------ |
| **Latency:** `/goal`, `/debrief` p95 < 250 ms at 1 k concurrent players |        |
| **Availability:** 99.5 % monthly                                        |        |
| **Security:** Audio clips encrypted at rest (KMS) + signed URLs         |        |
| **Compliance:** GDPR/DPDP export includes goals & debrief data          |        |

---

### 8  Data & Schema

```sql
TABLE session_goals        -- on /goal
  session_id UUID PK
  player_id  UUID
  focus_skill TEXT
  focus_reason TEXT
  focus_conf_pre INT      -- 0-100
  focus_expect INT        -- 1-7
  ts TIMESTAMPTZ

TABLE session_debrief      -- on /debrief
  session_id UUID PK
  focus_exec INT          -- 0-100
  focus_conf_post INT     -- 0-100
  session_difficulty TEXT -- easy|right|hard
  reflection_audio_url TEXT
  tags TEXT[]             -- NLP keywords
  ts TIMESTAMPTZ
```

**Derived columns**

```python
confidence_gain = focus_conf_post - focus_conf_pre
goal_fidelity   = focus_exec        # direct %
```

---

### 9  UX Wireframe Anchors (Figma #Links)

* **GS-Intro-Modal** – progress bar, Start button.
* **GS-Question-Page** – full-height Likert/slider, 30 s timer top-right.
* **Debrief-Slider Page** – confidence Δ slider + difficulty vote.
* **Debrief-Audio Page** – mic button with circular countdown.
* **Coach Grid Column** – “Goal Fidelity” + tooltip breakdown.

---

### 10  MVP vs Later

| MVP (Sprint +1 & +2)       | Later (Q3)                                        |
| -------------------------- | ------------------------------------------------- |
| Fixed list of focus skills | Player-defined custom skills with tag suggestions |
| Single audio insight       | Optional text or emoji journal                    |
| 0.5 s timer step           | Dynamic step (0.25–1 s) by exponential decay      |
| Weekly CSV for psych       | Integrated psych dashboard                        |

---

### 11  Success Metrics

| Metric                           | Baseline | 30-day Target |     |       |
| -------------------------------- | -------- | ------------- | --- | ----- |
| Goal-Setter completion rate      | 0 %      | ≥ 95 %        |     |       |
| Quick-Debrief completion rate    | 0 %      | ≥ 90 %        |     |       |
| Avg                              | CGS      |               | ±15 | ≤ ±10 |
| “About right” difficulty votes   | n/a      | ≥ 70 %        |     |       |
| Bug-free sessions (goal/debrief) | n/a      | ≥ 99 %        |     |       |

---

### 12  Timeline (2-Sprint capsule)

| Sprint         | Milestones                                                                                                |
| -------------- | --------------------------------------------------------------------------------------------------------- |
| **+1 (2 wks)** | DB migration • `/goal` API • Goal-Setter modal (Likert, slider, dropdown) • Unit tests • Cypress e2e      |
| **+2 (2 wks)** | `/debrief` API • Audio capture & upload • Adaptive rule integration • Coach grid column • Weekly CSV cron |

---

### 13  Dependencies / Risks

| Dependency                   | Risk                | Mitigation                                                         |
| ---------------------------- | ------------------- | ------------------------------------------------------------------ |
| Audio upload storage         | S3 latency spikes   | Pre-signed URL + local retry queue                                 |
| NLP tagger for reflection    | Low accuracy        | Use basic keyword & sentiment v1; replace with fine-tuned model Q3 |
| Player fatigue (extra 2 min) | Skip / rush answers | 180 s hard cap + gamified progress bar                             |

---

### 14  Open Questions

1. Should confidence slider be hidden from coaches to avoid bias?
2. Audio language—allow Hindi/vernacular or enforce English for NLP ease?
3. Do we auto-surface top focus skills to analysts for content marketing?

*(Owner: Devansh—decision by Sprint +1 planning.)*

---

**Verdict:** Green-light. Adds <2 min friction, unlocks bias calibration, and feeds adaptive engine with human signal. Ship in the next two sprints or cut it entirely.
