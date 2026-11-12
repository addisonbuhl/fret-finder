# Fret Finder: project brief

(Formerly "Ear to Fret". Saved data moved from `etf.*` to `ff.*` localStorage keys automatically on first load.)

Personal guitar ear-training app for Addison. The app plays a note (or a short run of notes), and Addison plays it back on guitar. The app listens through the phone mic, checks the pitch and tries to tell which string was used.

The current code is `index.html` in this project: a single self-contained file with no build step and no dependencies apart from Google Fonts.

## Goals (as Addison stated them)
- A phone app for personal use, run from the phone's browser and added to the home screen.
- It plays a note, and you play that exact note back (same pitch and octave).
- It detects which string the note was played on, if possible.
- String options: any string, a string you choose, or the app chooses the string.
- A custom fret range, e.g. only frets 6 to 12.
- Clear feedback when the note is right but in the wrong octave, or right but on the wrong string.
- Multi-note runs.

## Current features (v3.2)
- **Games:** (In Intervals, playing the reference note is ignored, not counted as a miss.) "Play it back" (hear notes, play them back) and "Intervals": the app shows and plays a reference note (with its string and fret), then plays a second note up to an octave away. You find and play the second note. Feedback names the interval, e.g. "You played E3, a major 3rd up from C3. Go higher." The reference shows as a filled brass dot on the fretboard. Notes per round is hidden in Intervals.
- **Modes:** Any string, One string (E A D G B e chips), and Random string (each note gets its own string).
- **Fret range:** separate low and high fret selects, each from open to 15. Frets outside the range are dimmed on the fretboard.
- **Notes per round:** 1 to 5. A run plays as a phrase with 0.8 s between notes, and each step moves a 5th (7 semitones) or less. You play the notes in order, and the app grades each one before moving to the next. If you miss a note, you retry that note; the run doesn't restart.
- **Feedback:**
  - Wrong note: "You played F♯3. Go higher."
  - Right note, wrong octave: "Right note (G), but one octave too high." Two or more octaves are counted too.
  - Right note and octave, wrong string: "…but you played it on the G string (2nd fret). It should be on the D string (7th fret)." A "count it" button overrides a wrong guess, and the app saves that sample to learn from.
  - In a run, each message starts with "Note 2:" and so on.
- **Other controls:** Play again, Reveal (shows names and positions, and breaks the streak), and Next (skip).
- **Options (checkboxes, all saved):**
  - Show note names.
  - Natural notes only: no sharps or flats. If the range has no natural notes, it falls back to all notes.
  - Focus on weak spots: notes you miss more often are picked more often (weight 0.4 + 3 × (misses + 0.5) / (plays + 2), so unplayed notes sit in the middle).
  - Play twice: the round (run, single note, or reference + note) plays twice, 1.5 s apart, before listening starts.
- **Saved progress:** stored in the browser and kept across reloads. The header shows the current streak and today's first-try count. Tap "Progress ›" to see today and all-time first-try rates, current and best streak, a 14-day bar chart, and the notes you miss most (shown after 3 plays). "Reset progress" asks for a second tap. A note counts as missed if you played a wrong note or wrong string before getting it, revealed it, or skipped it after a miss. Skipping a note you didn't try doesn't count against it. Up to 120 days of history are kept.
- **Live display:** the note being heard, a cents meter (green within ±10¢) and the string guess.
- **Fretboard SVG:** target rings (numbered in runs), a dot showing what you played (green if right, red if wrong), and dashed rings for every possible position when the string isn't known.
- **Tuner:** a "Tuner" button next to Calibrate opens a chromatic tuner: note, Hz, cents needle (green within ±5¢), "Flat, tune up" / "Sharp, tune down", and the matching open string highlighted. Tap a string chip to hear its reference note. Nothing is graded while it is open.
- **Mic level and sensitivity:** under the live display. The bar turns green once a sound is loud enough to count (gold line). The sensitivity slider (1 to 10, default 7) moves that line. It was added because an unplugged electric guitar was too quiet to register.
- **Calibration:** pluck each string open, at fret 5 and at fret 12 (18 notes). It's stored in the browser, so it survives reloads. "Clear all" asks for a second tap.
- **Settings** are remembered in localStorage.

## Fixes in v2.1
- "Count it" button now disappears as soon as another note is graded, so it can't save a wrong note as a string sample.
- Harmonic profile averaging counts each harmonic separately (missing harmonics no longer skew the average).
- Any-string mode picks each pitch equally often (notes found on two strings were picked twice as often).
- Mic reconnects when you come back to the app after the phone cut it in the background.
- Saved settings are checked on load, so bad values fall back to defaults.

## Fluency features (v4)
Addison's real goal is improvising fluency: playing what's in their head (or a song they hear) on guitar as easily as they whistle it. They don't know or want music theory, so the UI avoids theory words outside the older Intervals tab. The app should stay instrument-neutral where possible; only the fretboard picture is guitar-specific. The name "Fret Finder" is being replaced; no new name chosen yet.

- **Tabs:** Notes (the old "Play it back"), Echo, Songs, Hum, Intervals. Settings that only apply to some tabs hide on the others, and most settings live in a collapsed **Options** panel (string mode, custom frets, notes per round, checkboxes, mic sensitivity).
- **Echo:** the app plays a short tune and you play it back. Tunes start at 2 notes; a clean round (no misses, no reveal) adds a note, up to 8; two misses in a row at a length take one away (min 2). Saved as `progress.echoLen`. Tunes are built from one major key (picked at random from notes in range), mostly steps of 1 to 2 semitones, never more than 5, and end on the key's home note 60% of the time. Feedback says "close, go a little higher" instead of naming notes.
- **Songs:** 9 public-domain songs (Twinkle Twinkle, Mary Had a Little Lamb, Happy Birthday, Row Row Row Your Boat, Ode to Joy, Jingle Bells, Are You Sleeping, London Bridge, Old MacDonald), stored as steps from the home note plus beats and lyrics, one syllable per note. Each song is moved to the key that keeps the most notes inside the neck area. The current line of words shows, with the syllable you're on underlined.
  - **Wait for me:** you hear the first note, then play the song from memory; it waits on each note. "Hear next note" plays the note you're on, "Show me" puts it on the fretboard (counts as a miss for that note).
  - **In time (rhythm mode):** first note, 4-click count-in, then the song runs at the chosen speed (50 to 104 beats a minute) whether or not you play. A note counts if it's the right pitch within ±40% of a beat (at least ±250 ms) of when it's due, measured from when the note started. The result is "You hit X of N notes in time", and the best score per song is saved.
- **Hum:** tap Record hum, hum or whistle up to 10 notes, and recording stops after 1.5 s of quiet (or 12 s). The pitch track (up to 2.4 kHz while recording, so whistles work) is split into notes at short silences, loudness dips (so repeated notes stay separate) or pitch moves held ~0.1 s. Then you play it on guitar: the first note can be in any octave, and each next note must move the same distance as your hum did (±0.75 semitone, since voices drift). The tune shows as a row of bars by height; "Hear it" plays what the app heard, and "Show me" puts the numbered notes on the fretboard. Background-noise learning is paused while recording. In tests with synthetic hums (low, high, whistle range, legato, repeated notes, slightly off-pitch), all 6 tunes were split into the right notes.
- **Neck areas (home base):** Home base (open–5), Frets 3–8, Frets 5–10, Frets 7–12, Whole neck, plus Custom frets. The next area unlocks after 15 first-try rounds out of your last 20 in the newest area (Notes, Echo and Intervals count). Locked areas show a lock in the Neck menu, and a progress line shows under it. Users who already had their own fret range keep it as Custom.
- **Speed:** every first-try round in Notes, Echo and Intervals is timed from the end of the app's playback (time per note shown after the ✓). With **Speed goal** on, a bar fills while you play. The goal starts at 3 s a note (Intervals 4 s, Echo 2.5 s), tightens 15% after 5 beats in a row (min 0.6 s), and eases 15% after 6 misses of it in a row.
- **Daily streak:** practice time counts between the notes you play (any gap over 15 s counts as 15 s). 5 minutes makes a day count. The header shows the day streak and today's minutes. The old "in a row" streak is still in Progress.
- **Progress sheet:** day streak, minutes today, best day streak, recent time per note, first-try stats, in a row, neck areas open, Echo tune length.
- New saved fields in `ff.progress`: `secs`, `bestDays`, `unlocked`, `zoneHist`, `speed`, `goal`, `goalRun`, `goalSlow`, `echoLen`, `songs`. New settings: `zone`, `songId`, `timed`, `tempo`, `speedGoal`.

## How it works
- **Reference sound:** a Karplus-Strong plucked string with fractional delay (accurate to under 1 cent) and a 5.2 kHz low-pass filter. The mic input is ignored while the app's own notes play, plus 0.35 s.
- **Background-noise removal (v3.2):** each frame's spectrum (newest 4096 samples, Hann, 8192 FFT) has 3× a learned background spectrum subtracted. The estimate averages quiet moments: it falls with a rate of 0.08 per frame, rises at 0.03, and doesn't rise while a note is tracked or while the app plays. Pitch is found by YIN on the autocorrelation of the cleaned spectrum (divided by the window's autocorrelation), and the gate and level meter use the cleaned level. On a clean note, precision is ±1 cent. With A/C-style noise louder than the note, about half the notes are caught and none are wrong. With quieter backgrounds, it catches 13 of 13.
- **Mic level meter (v3.2):** the gold line is fixed at 60%, and the bar shows the level relative to the threshold, so turning sensitivity up makes the bar bigger.
- **Tuner smoothing (v3.2):** it shows the median of confident readings (clarity ≥ 0.7) from the last 0.9 s that agree on the note, and holds for 1 s between plucks.
- **App note length (v3.2):** 0.8 s with T60 2.2 s (was 1.6 s and 4.5 s), so the mic listens again sooner.
- **Quiet input handling (v3.1):** pitch and loudness are read from a band-passed copy of the mic (55 Hz high-pass, two 1.8 kHz low-passes), which removes most mic hiss. String detection still reads the unfiltered mic. The room-noise estimate drops fast, rises only while no pitch is heard, and holds still while a note sounds. Before, it slowly crept up to the level of a quiet ringing note and then shut that note out. When YIN finds no dip under 0.12, it takes the first dip within 0.06 of the deepest one, which avoids reading an octave low on noisy input. In tests with recorded quiet plucks (note level about 3× the hiss), the old build detected 0 of 12 notes and the new one detected 12 of 12, all at the right pitch. Noise alone triggered nothing.
- **Pitch:** YIN on a copy of the newest 4096 samples, decimated 2×, with a 1024-sample window. The range is 60 to 1200 Hz, which covers drop D up past the 15th fret. In tests on synthesized notes across all strings and frets 0 to 15 at 44.1 and 48 kHz, it was within about 1 cent with no octave errors at normal levels.
- **Note grading:**
  - A note must stay stable for at least 170 ms before it's graded.
  - A ringing note is graded only once. It can be graded again after a new pluck (detected from a jump in loudness), a moment of silence, or a change of pitch.
  - The detector adapts to the room's noise level.
- **String detection:**
  - First, the app works out which strings can physically play the note (frets 0 to 19). If only one string can, the answer is certain.
  - Otherwise it compares tone colour. It takes the level of harmonics 1 to 8 relative to their average (8192-point FFT), averaged over the first ~130 ms of the note.
  - It compares that against the calibration profile for each candidate string at that fret, interpolated between frets 0, 5 and 12.
  - Samples saved with "count it" are also used, within ±2 frets.
  - A small penalty applies above fret 12.
  - The distance is normalised per harmonic, using the spread across the calibration data.
  - This is a best guess and can be fooled by pick position and attack. Calibration with your normal playing style makes the biggest difference.
- **Mic settings:** echo cancellation, noise suppression and auto-gain are all off. Screen wake lock is on while practicing.

## Hosting
The mic only works over https, and the Claude chat preview blocks microphones, so the file has to be hosted:
- **GitHub Pages:** https://addisonbuhl.github.io/fret-finder/ from the `main` branch of github.com/addisonbuhl/fret-finder. Pushing to `main` deploys in about a minute.

On the phone, open the link and choose Share, then Add to Home Screen. Calibration is kept as long as the site address stays the same.

## Known limits
- On iPhone, the app's playback can sound quieter while the mic is active, because iOS switches to record-and-play mode.
- Standard tuning only.

## Ideas backlog (not built yet)
- **Alternate tunings:** drop D, DADGAD, half-step down.
- **Chord recognition:** play a chord and identify its root or quality. Much harder to build.
