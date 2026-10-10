# DSP: delays, modulation and feedback loops

For anything with a delay line: echo, chorus, flanger, vibrato, tape, BBD, early digital. Chorus, flanger and vibrato are the same machine as a delay (a short modulated tap), so everything here applies. Read `dsp.md` first.

## Feedback loops

- **Oversample nonlinearities inside loops.** A saturator in a feedback loop re-sharpens its own edges every lap, and without oversampling the edges alias into audible "digital" ticks. Even cheap 2× (midpoint interpolation, nonlinearity at both points, 2-tap average) fixed it in the oil-can delay.
- **Cap the loop bandwidth.** Short delays with high feedback put many laps per second through the saturator; a gentle roll-off in the feedback path (about 6 kHz) keeps runaway edges from building.
- **Map where it runs away, on a grid.** When two controls feed the same loop (feedback + a second recirculation), their gains add. Chart a grid of knob values against "decays / runs away" before the human tries it, and tune until the edge sits where it is playable (several knob positions wide, not a sliver).
- **A noise floor in a loop makes any loop gain above 1 self-start, at any frequency.** Hiss (or a denormal guard, or any input) seeds the loop, so an in-loop filter peak a little above unity runs away with no input at all. The BBD echo at Duration 7.7 ran away from its own hiss even though its DC loop gain was under 1. A compressive clip can't make "soft playing stays stable, hard playing tips it over": that needs an element whose gain rises with level (in tests).
- **A "Safety" output limiter** (lookahead 2 ms, soft knee, wet output only) lets runaways stay wild inside the loop while the output stays under a set ceiling.

## Delays and modulation

- **A modulated delay tap bends pitch by the rate the delay changes.** A fixed percentage of delay time gives far more pitch wobble on long delays than short ones. Scale speed-type modulation by current speed so the pitch deviation stays constant across delay times (the oil-can delay: from 20 cents of seasick wobble to a steady 4.5 at any time).
- **Varispeed:** let the read point follow an integrated motor speed with inertia (a one-pole on speed), and the pitch bends like tape. A fixed-cell "disc" model costs far more at short times than a tape-style delay line.
- **Hermite (4-point) interpolation** for audible taps; linear is fine for internal taps that are filtered anyway.

## Clocked delays (BBD, early digital)

Some originals don't move a tap: they change the clock that walks a fixed-length memory (bucket brigades, 1980s sampler-style delays). Then delay = length ÷ clock, sweeps bend pitch for free, and a frozen buffer varispeeds when the clock changes. Model it that way:

- A fixed buffer read and written at a **virtual clock**. Per host sample, count the ticks that fall inside it; interpolate the input at each tick's exact time, and **integrate the zero-order-hold output over the host sample** (a box filter) so a clock faster than the host rate decimates cleanly.
- Keep the original's fixed anti-alias and reconstruction filters at the host rate. When the clock runs slow, a little aliasing gets through, as on the hardware.
- The digital delay: 8-bit memory, 13:1 clock sweep, delays within 2 % at 44.1, 48 and 96 kHz, up to about 12 ticks per host sample, about 0.4 % of one x86 core at its fastest clock.

## Companders (NE570 and similar)

Put the compressor's level detector on its **output** and the expander's on its **input**. Then the expander sees the same envelope the compressor applied and the pair cancels, even during attacks, apart from what happened in between (quantisation, clipping). Sensing the compressor's input instead overshot every attack by about 16 dB in the digital delay's first model.
