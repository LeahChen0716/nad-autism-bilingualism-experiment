# nad-autism-bilingualism-experiment

Online experiment on **non-adjacent dependency (NAD) learning in autistic adults** (verbal vs. non-verbal), UCL MA dissertation.

Every participant completes **two structurally identical NAD target-detection tasks**: one with **spoken pseudowords**, one with **musical instrument sounds** in a counterbalanced order, plus background and post-task questionnaires. Built in [jsPsych](https://www.jspsych.org/) 7.3.4, hosted on [Pavlovia](https://pavlovia.org/), recruited via Prolific.

- **Live study:** `https://run.pavlovia.org/LeahChen/nad-experiment/`

---

## Repository contents

| Path | What it is |
|---|---|
| `index.html` | The **entire experiment** — jsPsych setup, the custom NAD trial plugin, all questionnaires, both tasks, and the Pavlovia integration (embedded inline). Self-contained; all libraries load from CDN. |
| `audio_verbal/` | 32 spoken-pseudoword files (`.wav`): 2 A, 2 B, 4 filler, 24 X. |
| `audio_nonverbal/` | 32 musical-tone files (`.wav`): flute A/B, flute fillers, violin two-tone X sequences. |
| `README.md` | This file. |

**Audio spec (both sets, identical):** mono, 44.1 kHz, 16-bit WAV, peak-normalised to −1 dBFS, identical processing so low-level acoustic differences cannot confound the verbal/non-verbal comparison.

---

## How to reproduce / run it

The experiment is a single self-contained `index.html` plus the two audio folders. There is **no build step**.

### On Pavlovia (the live setup)

1. Create (or open) a Pavlovia experiment backed by a GitLab repository.
2. Put these in the **repository root**, with **exact, case-sensitive names** (the server is Linux):
   ```
   index.html
   audio_verbal/      (32 .wav)
   audio_nonverbal/   (32 .wav)
   ```
3. Commit. In the Pavlovia dashboard, set the experiment status to **Piloting** (or **Running**). Toggling **Inactive → Piloting** forces Pavlovia to regenerate its config if needed.
4. Run via `https://run.pavlovia.org/<user>/<project>/`. The Prolific ID is read from the URL: `...?PROLIFIC_PID=xxxx`.

> If audio fails to load, it is almost always a name/case mismatch: the folders must be exactly `audio_verbal` / `audio_nonverbal` and every file name must match what `index.html` requests.

---

## Experiment flow

1. Information sheet & consent (with a decline option).
2. Eligibility / audio check (hearing; headphone confirmation, loops until confirmed).
3. **All task audio is preloaded once, up front**, while the participant answers the questionnaires (so there is no loading pause before each task).
4. Background questionnaires (autism diagnosis + details; language-acquisition history, per language).
5. A short questionnaire on everyday social preferences. *(Which background scale(s) to include is still being decided.)*
6. **Two NAD tasks, verbal and non-verbal, order counterbalanced**, separated by a short break.
7. Post-task awareness assessment (Dienes & Scott, 2005 style).
8. Debrief and return to Prolific. *(Debrief screen and Prolific redirect still to be added.)*

---

## The NAD task

- **Trial:** a three-element **A–X–B** sequence. The participant responds to whether the clip **ends with the target** — key **F** if it does, **J** otherwise.
- **Dependency:** during training, A predicts B across a variable middle element X. Per task: **2 A, 2 B, 24 X, 4 filler**.
- **Instructions are deliberately neutral** — participants are told only the response rule (respond to whether the clip ends with the target). They are *not* told to ignore the first element, and *not* told to look for a rule, so the dependency is learned **incidentally/implicitly**, as in the source paradigm.
- **Reaction time** is measured from the **onset of the third element** and may be **negative** (anticipation); a custom WebAudio-scheduled plugin handles the timing.
- **Blocks:** training → one **disruption** block (A replaced by a filler, removing the dependency) → one **recovery** block (dependency restored).

### Counterbalancing

- **Task order (verbal-first vs. non-verbal-first):** assigned by **Prolific-ID parity** — the sum of the ID's character codes; even → verbal first, odd → non-verbal first. This is deterministic and ~50/50 (mirroring the even/odd-ID assignment in Dumont et al., 2025). **With no Prolific ID (local testing), it falls back to random.** The method used is stored in the data as `task_order_method` (`prolific_id_parity` or `random_no_id`).
- **Within each task:** which A predicts which B, and which B is the target, are randomised per participant and stored in the data.

### Current parameters

All in one `CFG` block at the top of the task builder in `index.html`. Values follow **Dumont et al. (2025)**:

| Parameter | Value |
|---|---|
| Training blocks | 6 |
| Trials per block | 30 (12 target / 12 non-target / 6 filler) |
| Inter-element interval | 1000 ms |
| Response window | 750 ms |
| Practice trials | 8 (with feedback) |
| Response keys | F (target) / J (other) |

> Block count and trials-per-block are one-line edits in `CFG` and apply to both tasks at once.

---

## Data

Per NAD trial: `task`, `block_type` (practice/training/disruption/recovery), `block_index`, `trial_role` (target/nonTarget/filler), `a_id` / `x_id` / `b_id`, `is_target`, `response`, `rt` (from B onset; can be negative), `correct`.
Per participant: `prolific_id`, `task_order`, `task_order_method`, and per-task counterbalancing (`<task>_pairing`, `<task>_target_b`, `<task>_a_for_target`).
Questionnaire and awareness responses are tagged by `screen`.