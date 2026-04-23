# Star power

⭐⭐⭐⭐⭐

An Arduino-based gadget
that flashes rainbow lights on a short LED strip
and plays triumphant chiptune loops
triggered by a single button.
Inspired by classic arcade power-up sequences.
Build one and install it in your car
to give your ride a power-up.

⭐⭐⭐⭐⭐


## Hardware

- Arduino-compatible board with I2C and SPI
  (developed on a SparkFun RedBoard / Uno-class part)
- [SparkFun Qwiic MP3 Trigger](https://www.sparkfun.com/products/15165)
  with a microSD card for audio files
- APA102 / DotStar LED strip, 60 LEDs
- Momentary pushbutton (wired to GND, uses internal pull-up)

### Wiring

| Signal       | Pin              | Notes                          |
|--------------|------------------|--------------------------------|
| Button       | D8               | `INPUT_PULLUP`, other leg GND  |
| LED data     | D11              | APA102 data                    |
| LED clock    | D13              | APA102 clock                   |
| MP3 trigger  | SDA / SCL (I2C)  | Qwiic connector                |

## Build and upload

Dependencies (install via the Arduino Library Manager):

- `FastLED`
- `SparkFun Qwiic MP3 Trigger Arduino Library`

Then open `star_power/star_power.ino` in the Arduino IDE
(or use `arduino-cli`),
select your board, and upload.

## How it works

The sketch is a three-state machine:

- **IDLE**: strip dark, waiting for a button press.
- **SELECTING**: while the button is held,
  cycle through tracks every `cycle_duration_ms`.
  Each track plays a short loop and drives the LED gradient animation
  so the user can preview both sound and visuals.
- **PLAYING**: on release, commit to the current track
  and play for up to `total_duration_ms`
  (or until the button is pressed again).

## What this repository does **not** include: the audio files

To run the sketch end-to-end you must supply your own audio files.
A `tunes/` directory at the repo root is gitignored
as a convenient place to stash local copies;
only the files on the MP3 trigger's microSD card are read at runtime.

### Audio file contract

The Qwiic MP3 Trigger plays files by numeric index from its microSD card.
File naming convention used by the library:

```
/F001.mp3
/F002.mp3
/F003.mp3
/F004.mp3
/F005.mp3
```

The sketch calls `mp3_.playFile(track_index + 1)`,
so index `0` maps to `F001.mp3`, `1` → `F002.mp3`, and so on.

### Timing metadata

Each track needs two hand-measured values,
defined in `star_power.ino` in the `TRACK_SPECS` array:

- `cycle_duration_ms` — the length of one musical loop.
  The LED gradient completes two sweeps per cycle,
  so tune this until the animation feels locked to the beat.
- `total_duration_ms` — how long the track should play
  before auto-returning to `IDLE`.
  Usually the full clip length, or a clean cut point.

The values currently committed in `TRACK_SPECS`
were measured against the author's original five tracks
and will not be correct for your own audio.
Replace them with measurements of your own files
before expecting the animation to line up.

### Integrating your own tracks

1. Prepare one or more MP3 files
   that you have the rights to use.
   Short looped clips (roughly 10–25 s) work best
   with this state machine.
2. Copy them to the microSD card as `F001.mp3`, `F002.mp3`, … `F00N.mp3`.
3. For each file,
   measure the loop length (`cycle_duration_ms`)
   and the full playback length (`total_duration_ms`).
4. Update `TRACK_SPECS` in `star_power/star_power.ino`
   with one entry per file, in order.
   The number of entries determines the number of selectable styles;
   `NUM_TRACKS` is derived from the array size.
5. Optional: adjust `COLOR_SEQ` to taste —
   the gradient cycles through its entries in order.

## Repository layout

```
star-power/
├── LICENSE              -- MIT, code only (see scope note in file)
├── README.md            -- this file
└── star_power/
    └── star_power.ino   -- the sketch
```
