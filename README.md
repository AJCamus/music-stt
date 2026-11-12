# Music STT — a speech-to-text engine built for songs

Concept draft — Aaron Camus / Ember, 2026-11-05. Not a build plan yet, just the idea written down cleanly enough to share and to hand to Echo.

## The problem

Regular STT is built for speech, not singing. Songs break it in specific ways:

- Vocals buried in the mix — drums, bass, and guitars mask consonants.
- Words stretched across notes — one syllable can last a whole bar.
- Melisma — one word sung across several pitches, so the sound changes while the word doesn't.
- Slang, invented words, and names — lyric sheets are full of words a speech dictionary never sees.
- Timing matters — for captioning, *when* a word lands is as important as *what* it is.

CapCut's music-STT and general engines get close, then drift. The goal here is an engine that expects music, instead of treating it as noisy speech.

## The core idea: stemming + multiple passes

### Stemming as a hand-up

Don't make the first pass commit to an exact word form. Match the stem first, sort out the form later.

- "singing" / "sang" / "sung" → stem **sing**
- A stretched, melisma-heavy vowel still maps to the same stem as the clean spoken form.
- The stem narrows the candidate set early, so later passes aren't searching the whole dictionary — they're choosing between a handful of forms of the same root.

Stemming is a scaffold, not the answer. It buys the second pass room to work.

### Pass 1 — Comprehension (acoustic)

Raw transcription, but humble about it:

- Transcribe the vocal track (isolated vocals when available, full mix when not).
- Keep a shortlist of candidates per word, with confidence — never commit to a single guess at this stage.
- Keep timing: word-level timestamps, even rough ones, because the caption pass needs them.
- Flag low-confidence spots explicitly instead of papering over them.

Output of Pass 1 is a lattice: several possible readings per position, with timing.

### Pass 2 — Context (the song as dictionary)

Go back over the lattice using everything the song itself tells you:

- **Rhyme scheme** — if the line should rhyme, that kills most wrong candidates.
- **Meter and phrasing** — a word has to fit the beat and the phrase length.
- **Repeated sections** — choruses repeat. If the chorus is confident once, it anchors every other repeat.
- **Vocabulary already in the song** — words that appear elsewhere in the lyrics are more likely than new ones.
- **Stem consistency** — if Pass 1 heard "sing" in verse one, "singing" in the chorus is a better bet than a different root entirely.
- **Lyric-sheet alignment, when available** — known lyrics turn Pass 2 into forced alignment (matching known words to timing) instead of guessing.

Output of Pass 2 is the final transcript, with per-word timing and a confidence score that actually means something.

### Optional Pass 3 — Caption-ready check

A short final pass for the captioning use case:

- Word blocks that fit on screen — split at phrase boundaries, not mid-word.
- No block longer than the phrase it covers.
- Flag any word still below confidence for human review, rather than guessing silently.

## Training material

This is where Teach-by-Watching feeds in directly. The lyric-adjustment training video Aaron is recording (Nov 5) is exactly the right shape:

- Screen capture — what the human did.
- Voice narration — why: waveform cue, lyric anchor, split decision, and the reasoning in his own words.
- Lyrics sheet — ground-truth text.
- Isolated vocals — cleanest signal for timing.
- Full mix audio — real-world condition, music and all.

Paired audio + known lyrics is the training data: clean vocals teach timing best, the full mix teaches robustness. Label which is which.

More material, same shape, over time: different voices, different genres, different mixes. Mistakes and corrections kept in — a correction is training data, not a failed take.

## How you'd know it's working

Test against songs with known lyric sheets, and score:

- Word error rate, split by clean vocals vs. full mix.
- Timing error — how far off the word onsets are, in milliseconds.
- Confidence calibration — when it says it's unsure, is it actually wrong more often?
- Caption usability — how many blocks need a human fix before they're publishable.

Beat general-purpose STT on music, not on speech. That's the bar.

## Open questions

- Stemmer choice: a standard English stemmer, or one tuned for sung English (stretched vowels, dropped consonants)?
- How much does the isolated-vocal pass help vs. the full-mix pass — and can the engine learn to separate vocals itself when no isolated track exists?
- Where does the chorus-anchor logic live — in Pass 2, or as a pre-pass that finds repeats first?
- Licensing: training on Aaron's own songs and licensed lyric sheets first; what counts as fair training material for other people's songs?
- Scope: start with English, or design the stem/context passes so other languages can slot in later?

## Staging

1. Concept + training-material format (this sheet).
2. A small test set: a few of Aaron's songs, lyrics known, vocals + mix both available. Score a general STT engine on them first — that's the baseline to beat.
3. Pass 1 prototype: stem-aware candidate lattice, timing kept.
4. Pass 2 prototype: rhyme/meter/repeat rescoring on the lattice.
5. Caption-ready check, then a real caption job run through it end to end.

Repo is live — concept stage, no build started yet.
