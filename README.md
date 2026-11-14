# Headroom

A local-only browser tool for seeing where an ASV spends its pressure support
range, one slice per night.

Headroom reads ResMed `*_PLD.edf` files and lays each night out as a slice on a
long track you can move along in time. Each slice shows pressure support (PS)
against slew (how fast PS was moving), so you can see how a machine's servo
loop behaved across weeks and months, and how that changed when settings,
medications or anything else changed.

Hosted at [wobble-analysis-tool.xyz/headroom/](https://wobble-analysis-tool.xyz/headroom/),
alongside the other [Wobble Analysis Tool](https://wobble-analysis-tool.xyz/) companions.

Current version: **v0.1, experimental.** See [Caveats](#caveats) before
reading anything into it.

## Run

Open `index.html` in a modern Chromium-based browser. No install, build step,
server or network connection is needed; the plotting library is bundled.

- **Add a whole folder:** pick the SD card or its `DATALOG` folder. Only
  `*_PLD.edf` files are read; `BRP`, `EVE`, `SAD` and anything else are skipped
  without being opened.
- **Add files:** pick individual files. Batches add to what is already loaded,
  so a phone can go one day folder at a time.
- `STR.edf` is used for the PS limits in force each night if it is present.
- **Clear all** empties everything. Reloading the page also starts empty.

Nothing leaves your browser.

## What it shows

### The track

Each night is a slice standing on a long track. Left to right is PS (cmH₂O);
up and down is slew (cmH₂O/s), rising above the middle line and falling below
it. Darker means more of the night was spent there, from the machine's
2-second samples. Scroll, drag, use the arrow keys or press Play to move
forward and back in time. Zoom and rotation are fixed on purpose.

- **Orange lines** are each night's floor and ceiling (PS min and max), thicker
  the more of the night PS sat on them. Joined slice to slice they form the
  rails, so a settings change shows as a step.
- **The shape** (a kite) sums up each night with four corners: left and right
  are the 10th and 90th percentiles of PS; top and bottom are the 90th
  percentiles of rising and falling slew, each placed at the mean PS while
  rising or falling. Tall and narrow means quick, small corrections; wide and
  tall means large swings; a lopsided kite means it climbs differently than it
  falls.

### Marks, eras and the difference view

**Mark this night** starts a new era at the current night, with a label you
choose (a settings change, a new medication, a new mask). Marks are kept in
your browser.

**Difference from baseline** recolours every slice as the difference from a
chosen baseline era: teal where a night spent more time than the baseline,
violet where it spent less, blank where it matched. **Contrast** amplifies the
differences. The baseline era's shape is drawn dashed for comparison.

The **Eras** strip pools each era into one slice, side by side.

### Numbers per night

| Name | Meaning |
| --- | --- |
| At floor / At ceiling | Share of the night PS was within a tolerance (default 0.2 cmH₂O) of its min or max |
| PS travel | Total PS movement, up and down added together, per minute |
| Average one-way run | Average time PS kept moving in one direction before reversing |
| Shape corners | The kite described above |

### Exports

- **Save image (PNG):** the selected night's slice, rails, shape, axes and
  numbers in one picture, in the current view.
- **Save numbers (CSV):** the selected night as one row.
- **Save all nights (CSV):** one row per night, including era labels and all
  shape corners, for charting elsewhere.
- **Save eras image:** the era strip.

## Caveats

This is an experimental, personal project shared in case it is useful to
someone else. Please read these before relying on anything it shows.

- **Not medical advice.** Headroom is not a medical device and is not meant
  to diagnose, treat or guide therapy. Do not change PAP settings or
  medication because of it; talk to a qualified clinician.
- **Not affiliated with ResMed.** File formats are read from public knowledge
  of ResMed's EDF output and may differ between models and firmware versions.
- **Validated lightly.** The reader has been checked against a small number of
  real AirSense 10 sessions and synthetic files. Other machines, firmware or
  modes may parse wrongly or not at all.
- **ASV only.** Sessions where PS barely moves (spread under 0.3 cmH₂O
  between its 1st and 99th percentiles) are treated as fixed pressure support
  and skipped.
- **Rails can be estimates.** Without `STR.edf`, the floor is the lowest PS
  seen that night and the ceiling the highest PS seen within a week either side
  of it. This blurs settings changes by about a week. You can type the real
  limits under Rail settings.
- **Coarse time resolution.** Slew comes from 2-second samples, roughly half a
  breath. It shows how the servo moved across breaths, not within one.
- **No verdicts.** Resting on the floor through stable breathing is normal for
  ASV. The tool shows positions and shapes; what counts as good or bad for a
  particular person is not established here.
- **Experimental means it will change.** Metrics, defaults and the layout may
  change between versions, so numbers from different versions may not compare.

## Files

- `index.html`: the whole tool (parsing, metrics and drawing).
- `plotly-2.35.2.min.js`: [Plotly.js](https://github.com/plotly/plotly.js)
  2.35.2, MIT licensed, used for the per-night plots.

Keep EDF recordings out of the repository; `.gitignore` excludes them.

## License

MIT. See [LICENSE](LICENSE). Plotly.js is MIT licensed by Plotly, Inc.
