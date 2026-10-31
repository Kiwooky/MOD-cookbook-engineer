# DSP: drive and saturation

Read `dsp.md` first.

## Saturation and drive

- **Oversample nonlinearities that work hard.** A fuzz or high-gain drive makes harmonics far above the input; past half the sample rate they fold back as inharmonic "fizz". 2× (or more for fuzz) with a decent filter either side; a gentle clip at low gain may not need it. Test: a 3–5 kHz sine at full gain shouldn't produce tones below it.
- **Enough gain to reach the curve.** A guitar peaks around −20 dBFS (0.1), and a soft clip (tanh and its rational stand-ins) is nearly straight below about 0.5, so the drive has to bring the signal to 1–3 to saturate. Typical Drive ranges: a boost up to ~20 dB, an overdrive ~30–40 dB at the top, distortion and fuzz ~40–60 dB. Map the knob exponentially, e.g. `gain = powf(10.f, knob01 * 2.f)` for 0–40 dB, so each part of the travel does something. Crossroads mapped 0–100 % to 1–6× (16 dB): at full Drive a guitar came out barely past clean (2.5 % THD), while every loudness and alias test passed. Optional for the sound: a small DC bias before the clip (`softClip(g * x + b) - softClip(b)`) for even harmonics, and a high-pass of ~100–150 Hz before it so the lows don't turn to mud at high gain.
- **The clipping curve is the sound.** No limiter or knee after it. Set default Gain and Level so the effect at defaults is about as loud as bypass (average level within ±3 dB; peaks don't tell you, since clipping squashes them), and let the Level knob boost. A footswitch that jumps 15 dB at defaults is the first thing a player notices.
