# Harvesting lessons from other sessions

This playbook started from a hall reverb's handover and an oil-can delay build. An early digital delay was harvested in 0.2.0, a bucket-brigade echo in 0.3.0. Other plugin sessions (yours) will have learned things that belong here.

## What to look for

- **Rules that cost a round trip:** a build or upload that failed, a behaviour on the device that tests didn't catch, a MOD quirk.
- **Patterns that worked:** a DSP technique, a test, a face trick, a way of asking the human.
- **Tools** that would help every plugin (generalise them; don't paste plugin-specific scripts).
- **Corrections:** anything here that turned out wrong.

## A prompt to paste into an old session

> I'm building a shared MOD plugin playbook for the community (any LLM agent should be able to use it). Look back over this whole session and list everything we learned that would help someone building a different plugin: build/upload problems and their fixes, MOD/mod-ui quirks, DSP patterns, test techniques, face (modgui) techniques, and ways of working that saved time. For each item give: the rule in one line, what happened that taught it, and whether it's verified (on hardware / in tests) or only believed. Leave out things specific to this one plugin's sound. Also flag anything we did that turned out to be a mistake. Write every rule in your own words and include no code copied from other people's projects (the playbook is MIT; describe the technique instead). Leave out anything I've marked private.

## Credit and licence

Contributions are MIT, like the rest (`CONTRIBUTING.md`). Credit each harvest in `CHANGELOG.md` with what it came from and who contributed it. Lessons from closed-source or commercial plugins are welcome: the rule and the incident, never the code.

## Before you push (whoever pushes)

Run `python3 tools/bump_links.py <new version>` and add that version to `CHANGELOG.md`. It stamps the playbook's GitHub links with `?v=<version>` and rebuilds `ALL-IN-ONE.md`. AI fetch services cache GitHub pages for hours; without a new `?v=`, people's AIs can read the old `AGENTS.md` after you've pushed the new one.

## How to add it

The playbook is public, and agents load it into their context, so every line costs other people tokens. Add only what helps a different plugin, once, in the file the right reader opens.

- **Route by trait.** A rule goes in the file for the effects it applies to, never in a general file "just in case":

  | The lesson is about | It goes in |
  | --- | --- |
  | Every plugin (real-time, CPU, outputs, bypass, versioning) | `dsp.md`, `lv2-and-mod-rules.md` |
  | Delay, chorus, flanger, vibrato, feedback loops | `dsp-time.md` |
  | Reverbs | `dsp-reverb.md` |
  | Drive, fuzz, saturation | `dsp-drive.md` |
  | Envelope filters, compressors, gates, swells | `dsp-envelope.md` |
  | Pitch from a moving delay | `dsp-pitch.md` |
  | Faces | `modgui.md` |
  | Tests and test tools | `testing.md` |
  | Builder, recipes, repos | `path-a-single-recipe.md`, `path-b-github-repo.md` |
  | MOD-side bugs | `known-mod-issues.md` |

  A new kind of effect with three or more rules gets its own `dsp-<trait>.md`, a row in `effect-profile.md` and a line in `dsp.md`'s table (and in `tools/make_all_in_one.py`).
- **Then one line in `docs/lessons.md`**: the rule, the incident in a sentence, and the file it now lives in.
- **Merge, don't append.** If a rule is already there, sharpen it or add the new evidence; don't write a second version.
- **One or two sentences per rule,** with the number that proves it. No plugin-specific tuning, names of presets or sound design.
- **Keep claims honest:** "verified on a Duo", "in tests", "by ear" or "believed".
- **New tools** → `tools/`, generic, with a docstring, and a line in `testing.md` or the doc that uses them. Run `tools/check.sh` on `templates/plugin` afterwards to prove nothing broke.

## Writing straight into the playbook from another session

If that session can reach your local clone (a linked computer, or a shell), paste this there. It does the harvest and the edits; you review the diff and push.

> Harvest this session into my MOD plugin playbook. My local clone is at `~/Documents/GitHub/MOD-plugin-playbook` (ask me for access if you can't see it). First read its `docs/harvesting.md` and follow it exactly: it says what to look for and which file each kind of lesson goes in. Then:
> 1. List what this session learned that would help someone building a *different* plugin, each with the rule, the incident and how it's verified. Check each against the playbook first and drop anything already covered (or sharpen the existing line). Show me the list before editing.
> 2. Edit the files in my clone: route each rule by trait, merge rather than append, one or two sentences each, plus one line per rule in `docs/lessons.md`.
> 3. Add a `CHANGELOG.md` entry with the next minor version, run `python3 tools/bump_links.py <version>`, and run `tools/check.sh` on `templates/plugin` if any tool changed.
> 4. Commit in my clone (if you can't write there, deliver the commit as a git bundle as `docs/path-b-github-repo.md` describes). Don't push; tell me the commit and I'll push from GitHub Desktop.
