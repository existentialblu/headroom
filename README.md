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

Current version: **v0.4, experimental.** See [Caveats](#caveats) before
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
  We call it a kite. Strictly it's a four-corner glyph; it only becomes a
  geometric kite on nights when rises and falls mirror each other, which makes
  the lopsidedness the interesting part.

### Rise ÷ fall

The kite's rise speed divided by its fall speed. Above 1, PS climbed faster
than it backed off; below 1, it climbed more gently than it backed off.
Whether one is better is an open question. Early observations in one person
suggest a value somewhat below 1 went with steadier nights, which would fit
the idea that a high-loop-gain system is destabilized by overshoot, but this
has not been tested across people.

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

Shown as tiles for the selected night and exported as CSV columns:

| Column | Meaning |
| --- | --- |
| `date` | Night, as YYYY-MM-DD. Sessions starting before noon count toward the previous night. |
| `era` | Label of the era the night belongs to. "Start" before the first mark. |
| `hours_recorded` | Hours of pressure data. |
| `ps_min_limit_cmh2o`, `ps_max_limit_cmh2o` | PS floor and ceiling in force. |
| `limit_source` | `STR.edf` (machine settings), `observed` (estimated from the data) or `typed`. |
| `pct_night_at_floor`, `pct_night_at_ceiling` | Percent of the night PS was within the tolerance (default 0.2 cmH₂O) of each limit. |
| `ps_travel_cmh2o_per_min` | Total PS movement, ups and downs added together, per minute. |
| `avg_one_way_run_s` | Average seconds PS kept moving one way before reversing. |
| `kite_ps_p10_cmh2o`, `kite_ps_p90_cmh2o` | Kite left and right corners. |
| `kite_rise_at_ps_cmh2o`, `kite_rise_speed_cmh2o_per_s` | Kite top corner: where and how fast PS typically rose. |
| `kite_fall_at_ps_cmh2o`, `kite_fall_speed_cmh2o_per_s` | Kite bottom corner: where and how fast PS typically fell (speed as a positive number). |
| `rise_fall_ratio` | Rise speed ÷ fall speed. Blank if PS never fell. |
| `cloud_ps_mean_cmh2o`, `cloud_ps_sd_cmh2o` | Mean and standard deviation of PS. |
| `cloud_slew_sd_cmh2o_per_s` | Standard deviation of the rate of change: how vigorously PS moved. |
| `cloud_slew_skew` | Skewness of the rate of change. Negative: quick drops and gradual climbs. |
| `cloud_ps_slew_corr` | Correlation between PS and its rate of change. |
| `cloud_pct_still` | Percent of samples where PS did not move. |
| `cloud_entropy_bits` | Shannon entropy of the fixed grid below: how much of the plane the night covered. |
| `cloud_cells_used` | Number of fixed-grid cells the night visited. |
| `ps_step_cmh2o` | Smallest PS change the file can record, from the EDF header. |
| `pct_moves_single_step` | Of the samples where PS changed, the percent that changed by exactly one recording step. |
| `machine` | Serial and model/version codes (SRN, MID, VID) from the PLD header. |

### The cloud grid

**Save cloud grid (CSV)** exports each night's cloud itself, for comparing or
clustering nights elsewhere. The grid is fixed for every night: 0.5 cmH₂O of
PS by 0.1 cmH₂O/s of rate of change, with the rate bins centred on zero so the
middle bin holds samples where PS sat still. Long format, one row per night per
occupied cell, with `pct_of_night` summing to 100 per night. It is computed
from the raw 2-second samples, without the smoothing used in the picture.

| Column | Meaning |
| --- | --- |
| `date`, `era` | As in the main CSV. |
| `ps_bin_low_cmh2o`, `ps_bin_high_cmh2o` | PS range of the cell. |
| `slew_bin_low_cmh2o_per_s`, `slew_bin_high_cmh2o_per_s` | Rate-of-change range of the cell; positive is rising. |
| `pct_of_night` | Share of the night's samples in the cell. |

### Exports

- **Save image (PNG):** the selected night's slice, rails, shape, axes and
  numbers in one picture, in the current view.
- **Save numbers (CSV):** the selected night as one row.
- **Save all nights (CSV):** one row per night, including era labels and all
  shape corners, for charting elsewhere.
- **Save eras image:** the era strip.
- **Save cloud grid (CSV):** the cloud grid described above.
- **Save column guide (MD):** a plain-language description of every CSV
  column, the cloud grid file, the eras in the export and the caveats below.

### Using the CSV with an AI or a spreadsheet

The column names carry their units, and the column guide is written to be
handed over alongside the CSVs. The main CSV gives a model one row per night
to trend; the cloud grid gives it the full shape of each night to compare,
average by era or cluster. If you ask an AI model to analyse your
export, give it both files so it knows what each number means and what the
numbers can't tell it. Keep in mind it is reading the same experimental
measures; it can find patterns, but it can't make them clinical.

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
- **Machines and firmware can change the picture.** Different machines,
  or different firmware on the same machine, may log PS differently: holding
  it flat between corrections, or dithering it by one recording step. That
  can move stillness, the kite's tips and the cloud columns without any change
  in therapy. Check the `machine` and `pct_moves_single_step` columns before
  comparing across a hardware or firmware change.
- **Experimental means it will change.** Metrics, defaults and the layout may
  change between versions, so numbers from different versions may not compare.

## Files

- `index.html`: the whole tool (parsing, metrics and drawing).
- `plotly-2.35.2.min.js`: [Plotly.js](https://github.com/plotly/plotly.js)
  2.35.2, MIT licensed, used for the per-night plots.

Keep EDF recordings out of the repository; `.gitignore` excludes them.

## License

MIT. See [LICENSE](LICENSE). Plotly.js is MIT licensed by Plotly, Inc.
