# ARGUS Scope Contract


## 1. Use case

> A surgical procedure was recorded from several fixed cameras. Someone asks, in
> natural language, when something happened. ARGUS returns the ranked moments on the
> procedure timeline and the camera that best shows each one.

The asker is a person reviewing a finished recording — for training review, workflow
analysis, or finding a specific moment in 90 minutes of footage. They are not being
assisted during surgery.

---

## 2. What the system does

**At index time**, given one procedure:

- reads the pre-extracted frames of its camera streams, which share one timeline
- cuts the timeline into fixed-length clips
- represents each `(camera, clip)` pair as a vector
- writes those vectors to a searchable index

**At query time**, given one natural language string:

- represents the query in the same space
- searches the index
- merges adjacent high-scoring clips into segments
- returns a ranked list of segments, each with `[start_s, end_s]`, a camera, and a score

Times crossing any boundary are **seconds on the procedure timeline**. Clip indices and
frame numbers are internal.

---

## 3. What the system does not do

These are choices, not gaps.

| Not doing | Because |
|---|---|
| real-time or streaming | the unit of work is a query over a finished recording |
| understanding surgery | ARGUS retrieves moments; it makes no clinical claim of any kind |
| camera synchronization | MM-OR provides it; the timeline is given |
| video decoding | MM-OR ships frames; there is no decode stage to build |
| detection, tracking, segmentation, pose | not produced, not consumed, not evaluated |
| scene graph generation | annotations are ground truth for scoring, never a prediction target |
| cross-procedure retrieval | one index covers one procedure until v0.5 |
| audio, transcripts, depth, point clouds | present in the data, deliberately unused until v2 |
| open-vocabulary queries | v2; see §5 |


---

## 4. What the data actually is

Grounded in `data/MM-OR/MM-OR_processed`, take `001_PKA` as the reference:

- **Frames, not video.** `colorimage/cameraNN_colorimage-XXXXXX.jpg`, ~1 fps.
  One timepoint ≈ one second. A 95-minute take is ~5,700 timepoints.
- **Streams.** `camera01`–`camera05` (ceiling RGB-D), `trackercam` (detail view),
  `simstation` (the Mako robot's *screen*, not a view of the room).
- **Timepoint index.** `timestamp_to_pcd_and_frames_list.json` maps a timepoint to the
  frame number in each stream. This is the only alignment ARGUS needs.
- **Relation labels.** `relation_labels/NNNNNN.json` holds `(subject, predicate, object)`
  triples per annotated timepoint. 18 takes carry them, 44–3,466 timepoints each.
- **Phase labels.** `take_timestamp_to_robot_phase/*.json` gives a phase string per
  timepoint, ~9 distinct phases per take.
- **Partial annotation.** ~30% of timepoints are annotated, and *not* uniformly: gap
  sizes are 1 or 6, with the dense runs falling on the active parts of the procedure.

Two facts from this shape everything downstream:

**Rare positives.** Per take: `Hammering` 15, `Scanning` 3, `Cleaning` 5, `Suturing` 9,
`Sawing` 59 — against `CloseTo` 6,262. The queries the README advertises are the rarest
events in the data. ARGUS is a needle-in-a-haystack retrieval problem, and single-take
metrics on single-digit positives are noise.

**Label leak.** Robot phase labels are read off the `simstation` screen, and
`screen_summaries/` already holds them parsed. Indexing `simstation` would let phase
queries be answered by reading a monitor rather than by observing the room.

---

## 5. What counts as correct

Ground truth is derived from MM-OR annotations. Four calls, made now, that fix the
meaning of every number ARGUS will ever report:

**5.1 Query vocabulary is closed, and action-only.**
Queries are templated from the annotation vocabulary. Only *action* predicates are
eligible — `Sawing`, `Drilling`, `Hammering`, `Cutting`, `Suturing`, `Cementing`,
`Cleaning`, `Scanning`, `Calibrating`, `Preparing`, `Assisting`, `Manipulating`,
`Holding`, `Touching` — plus the robot phase strings. The spatial predicates `CloseTo`
and `LyingOn` are excluded: they describe geometry, not events, and they would dominate
the query set. Free-form queries are v2.

**5.2 Unannotated timepoints are unknown, not negative.**
Scoring happens on the annotated subset only. Because annotation density correlates
with activity, calling an unannotated timepoint a negative would manufacture false
positives out of unlabelled activity.
*Consequence, to be stated with every reported number:* metrics are measured over ~30%
of the timeline and are optimistic relative to retrieval over the full recording.

**5.3 `simstation` is excluded from the index.**
It is a monitor, not a view of the operating room, and it carries the phase label.
It returns in v2 as a deliberate multimodal signal, not as a silent shortcut.

**5.4 Metrics are pooled across takes, never reported per take.**
A predicate with 15 positives in one procedure cannot be measured in one procedure.

**Reported at every change:**

- ranking — Recall@k and mAP, per predicate and macro-averaged over predicates
- localization — temporal IoU against ground-truth intervals
- cost — index build time, index size on disk, query latency

Two references are reported alongside, or a score means nothing:

- a **floor**: random ranking, and a frequency-prior ranking
- a **ceiling**: an oracle that ranks using the annotations themselves

A representation change without these numbers is not a decision.

---

## 6. Accepted limitations

Stated up front so they read as scope, not as failure:

- metrics cover the annotated subset of the timeline, not all of it
- the query vocabulary is closed until v2, so "natural language" means templated
  natural language
- rare predicates will have wide confidence intervals no matter how good the model is
- one procedure per index until v0.5
- "the camera that best shows it" is whichever camera scored highest, not a
  view-quality judgement

---

## 7. Deferred, with a trigger

| Deferred | Decided when |
|---|---|
| clip length and stride | ADR, before the first index is built |
| index backend | ADR, before the first index is built |
| encoder | v0.1 takes the cheapest one that closes the loop; compared in v0.2 |
| reranking | v0.2, only if v0.1 metrics show it is the bottleneck |
| cross-camera duplicate suppression | v0.3, when multi-camera merging is evaluated |
| audio, transcripts, depth, `simstation` | v2 |
| open vocabulary | v2 |

---

## 8. Definition of done for v0.1

One command indexes one procedure. One command runs the query set against it and prints
ranking, localization, and cost numbers next to the floor and the ceiling.

Nothing about that sentence mentions a model. That is the point.
