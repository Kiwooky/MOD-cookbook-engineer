# DSP for MOD

MOD units are small ARM computers. The Duo (Cortex-A7, 32-bit) is the tightest; design for it and the Duo X / Dwarf follow.

This file holds the rules for **every** plugin. Then read only the files for your effect's traits (`effect-profile.md`):

| File | Read it for |
| --- | --- |
| [`dsp-time.md`](dsp-time.md) | Delays, echoes, chorus, flanger, vibrato, tape and BBD: anything with a delay line, a modulated tap or a feedback loop |
| [`dsp-reverb.md`](dsp-reverb.md) | Reverbs and other diffuse tails |
| [`dsp-drive.md`](dsp-drive.md) | Boost, overdrive, fuzz, distortion, saturation |
| [`dsp-envelope.md`](dsp-envelope.md) | Anything that follows the playing level: envelope filters, auto-wah, compressors, gates, swells |
| [`dsp-pitch.md`](dsp-pitch.md) | Pitch bends and swoops made with a moving delay (vibrato, Doppler, whammy-style dives) |

## Real-time basics

- **`run()` is real-time:** no allocation, I/O, locks or printf. Allocate everything in the constructor and clear it in `activate()`. Audio buffers (delays, reverbs, choruses, loopers) are sized for **96 kHz**, power-of-two with a mask.
- **Controls arrive with the first `run()`,** not before `activate()`. Snap every smoother to its target on the first run, then smooth.
- **Smooth anything that moves a gain, a filter or a delay tap** (one-pole, about 10 ms for gains, longer for delay times, which glide like tape).
- **Land smoothers on their target.** In float, `g += c * (target - g)` stalls once the step is below float resolution: the BBD echo's "dry at unity" measured 1 + 3e-5 with a 10 ms smoother. Snap to the target at the end of the block once within about 1e-4 (in tests).
- **Scale every time constant and delay length by the sample rate.** Test at 44.1, 48 and 96 kHz.
- **Denormals:** add a tiny constant (1e-18) in every feedback path, including recursive filters and envelope followers.

## CPU

- **No per-sample `pow`, `exp`, `log`, `sin`, `tanh`.** Use a rational tanh (`x(27+x²)/(27+9x²)`, flat beyond ±3), a parabolic sine for LFOs, tables for anything periodic.
- **Control rate:** recompute coefficients every 16 samples, smoothing in between. Retune filters only while a control moves.
- **Divisions cost** on the Duo (~14+ cycles). Hoist `1/x` to control rate where you can.
- **Measure, don't guess.** `tools/bench.py` renders the same audio through several plugins and prints relative cost; run it on the Duo build under qemu for Duo-like ratios. Only the Duo's own CPU meter gives absolute load. The oil-can delay's first estimate was wrong in both directions.
- **Run a band-limited sub-chain at a lower rate.** If everything after a steep low-pass sits below a few kHz (a dark reverb, a lo-fi echo), process it at half the rate (a quarter at 96 kHz) and interpolate back up. Upsample with linear interpolation plus two one-pole low-passes (about 5 kHz): plain linear interpolation leaves images that read as ticks. An underwater reverb went from 2.1× to 1.4× the hall reverb's CPU on the Duo build (in tests and the benchmark).

## Outputs

- **Only the converter clips.** Audio between plugins is floating point, so a boost above 0 dBFS reaches the next plugin intact; only the unit's last output hard-clips. So:
  - **Effects that can run away** (feedback loops, resonance, self-oscillation): a soft limit just under 0 dBFS on the **wet** path. The oil-can delay's runaway peaked at +3.8 dBFS and hard-clipped: "digital ticks" in 1.0.2.
  - **Gain and drive:** no limit. Say in the README that a boost at the end of a chain needs the Level knob, or it will clip the converter.
  - **Everything else:** no limit needed if the effect can't add level; test that it doesn't.
- **Opposite-polarity outputs:** a classic stereo trick (dry + wet on one side, dry − wet on the other) is wide in stereo but cancels the wet if both outputs end up in one mono input. Say so in the README, and recommend one output for mono rigs.
- **Level:** guitars into a MOD peak around −20 dBFS (the nominal input level isn't documented; this is from testing). Synths, keys and drum machines at line level are much hotter. Tune level-sensitive effects (drive, compressors, gates, envelope filters, sag) for the source the person will use, not for a hot test signal.

## Tone

- **A tilt control is a shelf, not a crossfade.** Fading between a low-pass and a high-pass of the signal lifted the mids at the centre and dipped them either side. A first-order shelf whose gain passes through unity at the centre is flat at noon and tilts cleanly. In tests (Tapestry).

## Stereo versions of a mono effect

- **Share the mechanism, split the randomness, link the detectors.** What's one part in the real thing (a transport, an LFO, dropout timing) stays shared, or the two sides chorus against each other. Noise is generated per side, so it has width (L/R correlation near 0). Compressors, auto-level and noise reduction listen to both sides (the louder side, or the mean power), so a hit on one side doesn't move the image. In tests (Tapestry Stereo).
- **Keep the left side on the mono plugin's random seed,** and the left output *is* the mono plugin with the same source in both inputs. That makes a strong test (`testing.md`).

## Recreations: resistor mixing networks

Switch matrices and mixers on old gear often join several taps through resistors onto one node. Model the node as a passive bus, not a sum: V = Σ(Gᵢ·Vᵢ) ÷ (ΣGᵢ + G_load), with G = 1/R and G_load the next stage's input. Engaging more sources then gets fuller but less than additive, as on the hardware; plain summing made the BBD echo's button combinations far too loud. Normalise one reference setting to unity and let the rest follow (in tests).

## Noise

- **Noise as a modulation source** (some "chorus" circuits are a noise diode, not an LFO): white noise → one-pole low-pass, scaled by √(2/a) (a = the filter coefficient) for unit variance, then a soft clip and a one-pole high-pass. It gives a band-limited random drift whose level doesn't depend on the sample rate. The BBD echo: 1.6 Hz low-pass and 0.5 Hz high-pass from the schematic, added to the BBD clock (in tests).
- **One Noise control, and 0 means none of any kind.** Tapestry's Hiss knob at 0 still left a gritty rasp on cheap formats: it was the tape's grain (modulation noise), not hiss. Putting all of it on one knob (0 clean, 1 grain only, 2 and up adds hiss, 10 authentic) kept the character for those who want it. By ear on a Duo.
- **Modelled hiss and hum are rarely wanted.** In the oil-can delay's first hardware test, hiss and hum were switched off within seconds. Players kept the *movement* (warble, wear) and dropped the dirt. Build clean; add character through movement and saturation.
