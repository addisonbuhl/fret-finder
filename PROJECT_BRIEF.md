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

## Current features (v3.1)
- **Games:** "Play it back" (hear notes, play them back) and "Intervals": the app shows and plays a reference note (with its string and fret), then plays a second note up to an octave away. You find and play the second note. Feedback names the interval, e.g. "You played E3, a major 3rd up from C3. Go higher." The reference shows as a filled brass dot on the fretboard. Notes per round is hidden in Intervals.
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

## How it works
- **Reference sound:** a Karplus-Strong plucked string with fractional delay (accurate to under 1 cent) and a 5.2 kHz low-pass filter. The mic input is ignored while the app's own notes play, plus 0.35 s.
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
- **Netlify Drop:** drag a folder containing `index.html` onto app.netlify.com/drop.
- **GitHub Pages:** an alternative.

On the phone, open the link and choose Share, then Add to Home Screen. Updating the app means re-uploading `index.html`. Calibration is kept as long as the site address stays the same.

## Known limits
- On iPhone, the app's playback can sound quieter while the mic is active, because iOS switches to record-and-play mode.
- Standard tuning only.

## Ideas backlog (not built yet)
- **Tempo and rhythm:** play the run back in time with a click.
- **Alternate tunings:** drop D, DADGAD, half-step down.
- **Chord recognition:** play a chord and identify its root or quality. Much harder to build.
