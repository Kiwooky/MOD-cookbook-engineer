# DSP: pitch from a moving delay

Bends, swoops, vibrato and Doppler made by moving a delay tap. The pitch ratio is 1 − (change in delay per sample): pitch only moves while the delay moves, and settles back to the played note. For a held interval (a steady octave down) you need a real pitch shifter (granular or phase vocoder), which this playbook doesn't cover yet. Read `dsp.md` and `dsp-time.md` first.

- **Going up costs latency.** A tap that speeds up has to start behind and catch up, so an upward swoop of ratio r over time T needs about (r − 1)·T of delay waiting in front of the note. At big settings the top of the swoop happens before the read point reaches the note: a requested two octaves up was heard as about +17 semitones, arriving late. Say so in the README. In tests.
- **Going down has no lag, but never below zero speed.** Size a downward swoop with (1 − 1/r), not (r − 1); otherwise large settings read backwards. Two octaves down = quarter speed. In tests.
- **A smooth contour: drive the delay from a two-pole envelope.** Two one-poles at T/2 each give a rise whose slope peaks at 0.736/T, so depth = (r − 1)·T/0.736 reaches ratio r at a full pluck, the same at every sample rate. In tests.
- **Filter before the bend, not after.** A deep dive moves the note out of a following band-pass and it vanishes. In tests and on a Duo.
- **Hermite (4-point) interpolation** on the moving tap; buffer for the largest depth at 96 kHz.

## Tests

- **Zero-crossing pitch tracking needs care:** skip silent windows (they read garbage), take the median of three windows (it spikes where two summed paths nearly cancel), and use longer windows for low notes (two octaves under 220 Hz needs about 60 ms). In tests.
- **Turn off anything that chirps** (dispersion, reverb) while measuring a swoop: any onset through a dispersive path reads as a pitch sweep.
