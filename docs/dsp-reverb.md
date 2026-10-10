# DSP: reverbs

Read `dsp.md` first, and the feedback-loop rules in `dsp-time.md` (a reverb tank is a feedback loop).

## Reverbs

- **Set the loop gain from the RT60 you want, not by ear.** For a tank whose energy passes *n* gain stages per lap of total length *L* seconds, each stage is g = 10^(−3·L / (n·RT60)). The hall reverb (Dattorro figure-eight, n = 4, L ≈ 0.73 s) maps its Decay knob to a target RT60 and measured slightly short of it (5.6 s for 5.9 s) because the damping filters take some energy too. In tests.
- **Scale every tank and diffuser length from the reference rate** (Dattorro's 29 761 Hz) by fs ÷ ref, including tap positions and modulation depth. The hall reverb's RT60 then stayed within ±0.1 s at 44.1, 48 and 96 kHz. In tests.
- **Fixed lengths as integer taps, modulated allpasses interpolated.** Only the tank's modulated allpasses (and any chorus or pre-delay) need fractional reads; the input diffusers and second allpasses can be whole samples. That's cheaper on the Duo; the hall reverb's RT60 moved by about 0.1 s when it switched (in tests; not compared by ear).
- **Drain the lows inside the loop to stop boom.** With a dark tank, only the lows survive and long tails boom. A one-pole high-pass (about 100 Hz) in each half of the tank makes the lows decay faster; the measured RT60 drops (a 10 s set point measured 5.7 s), so re-map the decay knob afterwards. In tests; by ear on a Duo.
- **Dispersion (a "spring" or underwater chirp): a chain of stretched first-order all-passes.** y = a·w + w[n−K], w = x − a·w[n−K], with K = 2–4 so the effect sits in the audio band. Lows are delayed most, so a pluck arrives as a falling chirp. Make the knob linear in each stage's low-frequency delay D: a = (1 − D)/(1 + D); a linear knob on `a` puts nearly all the change in the last quarter. 48 stages at D up to 16 gave lows about 37 ms behind highs, the same at 44.1, 48 and 96 kHz. In tests.
- **Not every reverb is a room.** A room tank alone sounded like a cave when the target was underwater. What made it read as water: a few separate bounces (alternating polarity, each darker) feeding a soft tank, dispersion, a narrow stereo image, and an inverted reflection whose delay drifts slowly. Research what makes the space distinct before tuning a tank. By ear on a Duo.

## Time controls read true

- **A time knob means the whole path.** Anything in front of a delay adds to it: the hall reverb's chorus has an 8 ms centre delay, so its 81 ms pre-delay first arrived at 89.9 ms. Subtract the upstream delay (clamped at 0) and test the onset end to end; it measured 81.9 ms afterwards. In tests.

## Lo-fi quantisation inside a loop

- **Round, don't truncate, when you quantise values that recirculate.** Truncating toward zero removes a little energy on every pass. The hall reverb's "Vintage" 16-bit tank storage first cut the decay from 5.5 s to 4.0 s (and to 3.4 s at 96 kHz, which has more passes per second). Rounding kept the decay time, and the very end of the tail still falls to exactly zero. In tests.
