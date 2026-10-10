# DSP: drive and saturation

Read `dsp.md` first.

## Saturation and drive

- **Oversample nonlinearities that work hard.** A fuzz or high-gain drive makes harmonics far above the input; past half the sample rate they fold back as inharmonic "fizz". 2× (or more for fuzz) with a decent filter either side; a gentle clip at low gain may not need it. Test: a 3–5 kHz sine at full gain shouldn't produce tones below it.
- **The clipping curve is the sound.** No limiter or knee after it. Set default Gain and Level so the effect at defaults is about as loud as bypass (average level within ±3 dB; peaks don't tell you, since clipping squashes them), and let the Level knob boost. A footswitch that jumps 15 dB at defaults is the first thing a player notices.
