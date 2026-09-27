# Urdu trainer — agent notes

```
Tatoeba + hermitdave FrequencyWords + wordfreq + kaikki Wiktionary (ur + hi twin) + Stanza (tagging)
        |
        v
tools/build_pack.py (packbuilder, langs/ur.py)  -->  pack/*.json (words, sentences,
        |                                              passages, script, attribution)
        v
engine/tools/jsonify_pack.py  -->  pack/*.js (generated)
        |
        v
build.sh (engine/app.html + engine/core.js + pack js)  -->  index.html + sw.js
        |
        v
GitHub Pages https://bannerless-studio.github.io/urdu/
progress lives in localStorage key vocab_ur on the shared origin
```

Why it is built this way: single-file site + service worker for offline;
engine as a git submodule so every language ships the same drills; pack ids
frozen (`tools/id_map_v1.json`) so learner progress survives rebuilds;
Urdu-specific rules (Nastaliq normalisation, Tatoeba text-quality layers,
Hindi-twin dictionary fallback, compound/context disambiguation, script
primer) live in `engine/tools/packbuilder/langs/ur.py`, not in this repo.
Stanza's ur model is CC BY-NC-SA 4.0: no model files or tagger output ship.

## Commands (pinned)

- Rebuild pack: `python3 tools/build_pack.py` (shim for
  `PYTHONPATH=engine/tools python3 -m packbuilder build --lang ur --repo .`)
- Rebuild script primer:
  `PYTHONPATH=engine/tools python3 -m packbuilder script --lang ur .`
  -> `pack/script.json`
- Rebuild passages (run the pack build first: needs
  `.cache/kaikki_ur.jsonl`, which `tools/build_pack.py` unpacks):
  `PYTHONPATH=engine/tools python3 -m packbuilder passages --lang ur .`
  (`--check` for a report only)
- Convert to JS: `python3 engine/tools/jsonify_pack.py pack`
- Build site: `./build.sh`
- Check (must pass before every commit of index.html): `./check.sh`
- Against a vocab-engine checkout other than the submodule: set
  `PACKBUILDER_PATH=../vocab-engine/tools`
- QA helpers: `PYTHONPATH=engine/tools python3 -m packbuilder {scan,sample} --lang ur --repo .`
- Engine tests live in vocab-engine (see its CLAUDE.md)

## Always

- Commit `index.html` and `sw.js` together; `check.sh`'s stale-build guard
  runs post-commit.
- Bump the engine submodule only to a vocab-engine main sha; rebuild after
  every bump.
- Keep ids append-only; never renumber (`tools/id_map_v1.json`).
- When a new Tatoeba sentence enters the shipped set after a rebuild, hand
  review it and add it to `tools/bad_sentences.txt` if it is ungrammatical
  or non-standard.
- Path-limited commits: `engine`, `index.html`, `sw.js`, `pack/`, `tools/`,
  `README.md`, `TODO.md`; never `.venv` or `.cache`.

## Never

- Edit `pack/*.json` by hand; change `tools/gloss_overrides.json`,
  `tools/gender_overrides.txt`, `tools/forced_a1.txt` or `langs/ur.py` and
  rebuild.
- Edit `pack/*.js`, `index.html` or `sw.js` by hand (generated).
- Delete `sw.js` (use `engine/sw.disable.js`).
- Add comments that say what the code does; only why, or an external
  reference.
- Push to main without `git merge-base --is-ancestor origin/main HEAD`.
- Ship Stanza model files or raw tagger output (licence CC BY-NC-SA 4.0
  forbids it).

## Forbidden patterns

- No renumbering `tools/id_map_v1.json` — learner progress across rebuilds
  depends on stable ids.
- No hand edits to any generated file (see below).
- No what-comments in code, only why/external-reference comments.

## Generated files

`pack/*.js`, `index.html`, `sw.js`, `pack/script.json`, `pack/passages.json`,
`tools/REPORT.md`, `tools/REPORT_passages.md` (manual QA sections inside
them are preserved across regenerations), `tools/id_map_v1.json` (frozen
once published — hand-edit never after that point).

## Where things are

`README.md` (end users), `tools/README.md` (builder inputs, file by file,
and the full Urdu-rules summary), `TODO.md` (residual defect classes + QA
history, incl. the engine-submodule pin status), `engine/` (submodule,
read-only here).
