# DSP: envelope-driven effects

For effects that follow the playing level: envelope filters, auto-wah, compressors, gates, slow-gear swells. Read `dsp.md` first.

## Followers

- **Two followers do most jobs:** a fast one (about 1 ms attack, 100–150 ms release) for "how loud is it now", and a slow one whose rise is the musical time control (a swell, a filter opening). The ratio slow ÷ fast, clamped to 1, is a slow-gear swell gain.
- **Clamp the detector input** (about 1.5× full scale after the sensitivity gain), or a hot input keeps the follower above its ceiling long after the note fades and the release starts late. In tests.
- **Sensitivity as a gain on the detector**, about 10^((knob − 50)/50) × 10, puts a guitar at −20 dBFS near full scale at the knob's centre.
- **Coefficients at control rate.** Retune a swept filter every 16 samples from the follower (a TPT state-variable filter takes this without zipper noise); smooth the follower, not the coefficients.

## Ranges

- **A knob that does nothing on one instrument may matter on another.** A width-only "Throat" control on an envelope filter did little on guitar and was removed, then came back as a centred control that slides the whole range up or down (±1.5 octaves), so bass gets a low setting. Level-match each end: the wide, low end measured 6.6 dB louder. In tests and on a Duo.
- **Offer the envelope as a switch** (Off / Fixed / Follow) when the effect is also useful without it. "Off" should mean off everywhere: every control the envelope drives goes dead, and the face can show which ones (`modgui.md`, status lights).

## Tests

- **Use harmonic plucks.** Pure-sine plucks (`pluck_train`) barely move an envelope filter, because there's nothing above the fundamental to sweep across. Use `harmonic_pluck_train` (`tools/lv2test.py`). In tests.
- **Measure at the source's real level** (`testing.md`): guitar peaks around −20 dBFS.
