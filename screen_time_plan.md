# Plan: Comparing Screen Time of Dr. Caitlin Lenox vs. Dr. Frost

**Show:** Chicago Med
**Seasons:** 10 and 11
**Method:** Subtitle / transcript analysis (proxy for screen time)

---

## 1. Goal

Estimate and compare the on-screen speaking time of Dr. Caitlin Lenox and
Dr. Frost across seasons 10 and 11 of Chicago Med, using subtitle and
fan-transcript data as a proxy. This avoids needing video files or automated
face recognition.

## 2. Caveats of the subtitle approach

Before starting, document what this method does and does not measure:

- **Measures:** speaking time (approximated from dialogue line counts / word
  counts).
- **Does not measure:** silent on-screen presence, reaction shots, background
  scenes, voiceover without attribution.
- **Attribution problem:** broadcast closed captions for Chicago Med rarely
  include speaker labels. Fan-transcribed scripts (e.g. Forever Dreaming,
  Springfield! Springfield!, 8flix) usually do. The quality of results depends
  almost entirely on finding a transcript source that labels the speaker.
- **Season coverage:** both S10 and S11 have fully aired as of April 2026,
  so full-season totals are available for both.

## 3. Data sources (in priority order)

1. **Fan transcript sites** that label speakers line-by-line:
   - `foreverdreaming.org` transcripts forum
   - `8flix.com` (screenplay-formatted)
   - `springfieldspringfield.co.uk` (usually unlabelled — fallback only)
2. **Official closed captions** pulled from Peacock / NBC, if accessible with
   a subscription. Requires speaker re-attribution via a diarization step,
   which defeats the simplicity of Option C — only do this if transcripts
   are unavailable.
3. **Fandom wiki episode pages** — occasionally include scene summaries
   naming characters, useful for spot-checks but not primary data.

## 4. Character disambiguation

Targets are confirmed:

- **Dr. Caitlin Lenox** — played by Sarah Ramirez, introduced in S10.
- **Dr. Frost** — confirmed as the intended character.

Note common misspellings and nicknames that will appear in transcripts
("Lenox" vs "Lennox", "Dr. Frost" vs "Frost" vs first name, etc.) and
define a regex that matches all of them.

## 5. Pipeline

### Step 1 — Acquire transcripts
- For each of S10 E01..E_last and S11 E01..E_last, download the best
  available speaker-labelled transcript into `data/transcripts/s{NN}e{NN}.txt`.
- Record the source URL and retrieval date in `data/sources.csv`.

### Step 2 — Normalise format
- Write a small parser that converts each transcript into a row-per-line CSV:
  `season, episode, line_index, speaker, text`.
- Normalise speaker names (uppercase, strip titles, collapse aliases) via a
  lookup table.

### Step 3 — Count
- For each episode, compute for each target character:
  - `line_count` — number of dialogue lines.
  - `word_count` — total words spoken.
  - `estimated_seconds` — `word_count / 2.5` (≈150 wpm average TV dialogue);
    cite the rate and keep it configurable.
- Emit `data/screen_time.csv` with one row per (episode, character).

### Step 4 — Aggregate & compare
- Per-season totals (minutes).
- Per-episode means and standard deviations.
- Percentage of each episode's total dialogue each character accounts for
  (controls for episodes that are simply longer or talkier).
- Paired comparison across episodes where both characters appear.

### Step 5 — Present
- A results table: season, character, episodes appeared, total minutes,
  mean minutes/episode.
- A bar chart per season (Lenox vs Frost).
- A per-episode line chart to show trends.
- A short narrative answering the original question with explicit uncertainty
  bounds.

## 6. Validation

- **Hand-check sample:** for 2 episodes per season, manually time the
  character with a stopwatch and compare to the word-count estimate. Report
  the ratio so readers can scale their confidence.
- **Sanity bounds:** neither character should exceed the episode runtime;
  flag any episode where estimates do.

## 7. Deliverables

- `data/transcripts/` — raw transcripts (respect source terms of use;
  if redistribution is disallowed, store only the derived CSV).
- `data/screen_time.csv` — per-episode counts.
- `analysis.ipynb` or `analysis.py` — the aggregation and plots.
- `results.md` — tables, charts, and the written conclusion with caveats.

## 8. Open questions to resolve before starting

1. Decide whether voiceover / intercom pages count as speaking time.
2. Pick the words-per-second conversion rate and document it.
