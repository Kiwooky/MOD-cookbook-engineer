# Testing

One command: `tools/check.sh` (see `AGENTS.md`). This page explains what it runs and what your plugin's tests should cover.

## The rig

| Step | Tool | Why |
| --- | --- | --- |
| Build native, Duo (arm32), Duo X/Dwarf (arm64) | `tools/harness.mk` runs the recipe's own Buildroot hooks without Buildroot | Same commands the builder runs; warnings fail |
| TTL vs DPF | `lv2_ttl_generator` + `tools/ttlcmp.py` | The hand-written TTL must match the code |
| Load check | `lv2info` | The bundle parses and the URI resolves |
| Audio tests, native | `tools/lv2host.c` + your test script (`tools/lv2test.py`) | DPF plugins won't load in `lv2apply`; this offline host does, with control changes at chosen samples (`@sample:port=value`) |
| Same tests on the Duo build | qemu-arm | Catches ARM-only problems; results must match native |
| CPU | `tools/bench.py` | Relative cost vs plugins you know; absolute only on the device |
| Face clicks | `tools/face_click_test.py` | Every control pressed from its default with mod-ui's own widget code: cleanly, with a 2 px wobble and (toggles) with touch taps; finds script-driven controls; runs the face script with mod-ui's semantics |
| Path-B package | `tools/package_harness.mk` | Builds the package `.mk` against a local copy of the repo, native and cross |
| Factory presets | `tools/presets_from_device.py` | Writes presets and reads every value back |

Packages: `g++ g++-arm-linux-gnueabihf g++-aarch64-linux-gnu qemu-user lilv-utils git make` and Python `numpy scipy rdflib` (plus `pillow playwright` for faces).

## What every plugin should test

Copy `templates/plugin/tests/test_simple_echo.py`. Each test is tagged with its trait (`docs/effect-profile.md`); keep `[every]` and the ones for your effect's traits.

Every plugin:

- **Timing / tuning** at 44.1, 48 and 96 kHz (the first echo lands on the Time setting, a filter's corner is where it should be, …).
- **Levels:** as the spec says (unity, or "+12 dB at Level max" for a boost); at default settings, average (RMS) level within ±3 dB of bypass, on a guitar-level signal. Peaks don't count: a drive squashes them.
- **Bypass:** dry at exactly unity, no step bigger than the signal's own.
- **Torture:** every knob at its extreme, hot input, three sample rates: finite.
- **No ticks** with musical input (`pluck_train` + `tick_count`).
- **Silence in, silence out** (unless noise is a feature).

By trait:

- **Tail:** bypass with Tails on (the tail rings out) and off (the wet fades).
- **Feedback loop:** torture at max feedback stays under 0 dBFS; the runaway grid.
- **Nonlinear / gain:** no aliasing at full gain (a 3–5 kHz sine makes no tones below it); gain at Level max as specified; default loudness against bypass (RMS).
- **Level-sensitive:** at the source's real level.

- **Reverbs:** RT60 (`rt60`) at all three rates (agreeing within a few tenths of a second), the decay knob monotonic over its range, pre-delay by onset (`onset_ms`), stereo outputs decorrelated (|corr| < 0.2), and the tail reaching silence.
- **First run:** render with a non-default value from sample 0 (pre-delay, bypassed) and check there's no glide or fade-in. Controls arrive with the first `run()`.

Then **one test per promise in the spec** ("Wobble 10 is about 4× stock", "Repeat 7 + Reverb 10 runs away, Repeat 6 never does"), and **one per hardware report**.

## Rules for writing tests

- **Test at the source's level** (`pluck_train` peaks around −16 dBFS, like a guitar; scale it up for line-level sources). The oil-can delay's Sag was tuned on hot test signals and barely reacted to a real guitar.
- **Smooth envelopes on test signals.** Abrupt starts and stops are clicks; the first tick hunt in the oil-can delay found the test signal's own edges.
- **Validate a detector before trusting it:** clean input must read 0, planted faults must be found. `tick_count` measures the *treble share* (clicks are almost all treble; bright notes are not), and was checked against planted clicks down to −46 dBFS, white noise, and clean plucks.
- **Test the knob's shape, not only its ends.** "Runs away at 10" passed while the real edge sat at 9.3, a sliver of the knob nobody could find. Test both sides of an edge at playable positions.
- **Measure delay times by cross-correlation** with a 20–50 ms noise burst, not by a threshold on the onset. Companders, filters and short delays smear onsets; the digital delay's 1.2 ms flanger setting read as "no echo" with a threshold.
- **Leave margin** in thresholds so native and ARM floating point don't flip a result.
- **Know where `tick_count` reads false.** Harmonic plucks with 2 ms attacks register as a tick on their own (−90 dBFS of treble), and so do real partials shifted an octave up past 12 kHz. Use 5 ms attacks, and check `tick_count(input)` is 0 first. In tests.
- **A steady sine through a reverb isn't a level reference.** The sine interferes with its own reverb at fixed phases, so its level moves several dB with settings. Compare settings against each other, or use harmonic or noise signals. In tests.
- **Compare ARM with native by envelope when the plugin accumulates a phase.** ARM GCC fuses multiply-adds by default (`-ffp-contract=fast`); in a clocked delay, a tick lands one sample differently after about half a second and sample differences reach 0.8. The BBD echo's two ARM builds matched each other to 1e-4 and native within 0.1 dB of 100 ms RMS (in tests).
- **rdflib: use `LV2['index']`, not `LV2.index`.** `Namespace` is a string, so `.index` is the string method; every port's index came back `None` (in tests).
- **To see what a unit's browser receives for a plugin**, build mod-ui's `utils/` (`libmod_utils.so`, needs `liblilv-dev` and `libjack-jackd2-dev`) and call `modtools.utils.get_plugin_info(uri)` with `LV2_PATH` pointing at the bundle: the exact port properties, scale points, presets and version mod-ui hands the face. It ruled out "the TTL flags don't reach the page" in one step on the BBD echo (in tests).
- **Make the gate fail on purpose once** (a wrong TTL default, an unused variable) to prove it can.

## The library (`tools/lv2test.py`)

`Plugin(order, defaults, n_in, n_out, atom_in=0, atom_out=0).run(x, sr, events, trace=None, transport=None, **controls)`, `check(name, ok, info)`, `done()`, and helpers: `db`, `rms`, `centroid` (brightness), `pluck_train` (pure sines), `harmonic_pluck_train` (24 harmonics, for filters and wahs), `tick_count`, `warble_cents` (pitch wobble of a steady sine), `max_step` (clicks). List output control ports in `order` too (any default); `run(..., trace='<symbol>')` returns `(audio, values)` with that output's value after every 256-frame block (`PB_TRACE_PORT` / `PB_TRACE_FILE` in `tools/lv2host.c`).

**Event ports** (tempo-synced plugins, MIDI out). DPF adds an atom input for `DISTRHO_PLUGIN_WANT_TIMEPOS` and an atom output for `DISTRHO_PLUGIN_WANT_MIDI_OUTPUT`, between the audio and the control ports; pass `atom_in=1` / `atom_out=1` so controls are numbered past them. `transport=dict(bpm=120, bpb=4, roll=(start, stop or None), every=False)` sends `time:Position` objects as a host would: by default only when the transport changes, so DPF extrapolates in between (the sparse case; `every=True` sends one per block). MIDI the plugin writes lands in `p.midi`, rows of (absolute frame, status, data1, data2). The looper's tests caught three real bugs this way (in tests).
