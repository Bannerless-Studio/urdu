# tools/ — Urdu pack builder inputs (dev/agent notes)

Purpose: everything the pack build reads or writes for the Urdu (`ur`) pack.
The actual build logic lives in vocab-engine's `engine/tools/packbuilder` and
`engine/tools/packbuilder/langs/ur.py`; this directory holds Urdu's
hand-maintained overrides plus the generated reports and frozen id map. Read
this before deciding what else in `tools/` you need to open.

## Hand-maintained (edit these to change the pack)

- `build_pack.py` — shim that runs
  `python3 -m packbuilder build --lang ur --repo .`. No Urdu logic here.
- `gloss_overrides.json` — hand gloss fixes, keyed `"lemma|pos"` (folded
  keys); covers the top 300.
- `gender_overrides.txt` — noun gender where the Urdu and Hindi dictionaries
  disagree or lack it. 124 lines; a `-` line leaves a noun unmarked.
- `forced_a1.txt` — A1 core list (closed grammatical sets live in
  `langs/ur.py`).
- `bad_sentences.txt` — hand-reviewed Tatoeba sentences to drop, keyed by
  normalised text (not Tatoeba id), 181 so far. A rebuild drops them; a new
  Tatoeba sentence that enters the shipped set needs the same hand review.
- `generated_sentences.tsv` — sentences written for this pack (marked
  `"src": "gen"` in `pack/sentences.json`); append-only, line order sets ids.
- `passages_src.json` — source text + rules for the 60 reading passages;
  follows the rules block of `persian/tools/passages_src.json`.

## Generated (do not hand-edit; rebuild instead)

- `id_map_v1.json` — frozen `"lemma|pos"` -> word id map. Keeps learner
  progress stable across rebuilds; append-only even before a schema change.
- `REPORT.md` — full build report (word-selection funnel, sentence stats,
  headword-spelling changes, CEFR cross-check). The manual QA/verdict
  section inside is preserved across regenerations.
- `REPORT_passages.md` — per-passage coverage numbers and QA notes for the
  Read tab passages.

## Not part of the pack build

- `requirements.txt` (packbuilder deps + Stanza) is installed once into
  `.venv`. Sources for the build itself (Tatoeba, FrequencyWords, wordfreq,
  kaikki Wiktionary ur + hi, the Stanza ur model) download into `.cache/`
  (gitignored) and are not tracked here. The passages build additionally
  needs `.cache/kaikki_ur.jsonl`, which `tools/build_pack.py` unpacks —
  run the pack build before the passages build.

## Rebuilding from scratch

```
git clone --recurse-submodules <this repo>
cd urdu
python3 -m venv .venv && source .venv/bin/activate
pip install -r tools/requirements.txt     # the Stanza ur model downloads into .cache/stanza on first build

python3 tools/build_pack.py               # pack/*.json + tools/REPORT.md
PYTHONPATH=engine/tools python3 -m packbuilder script --lang ur .     # pack/script.json (primer)
PYTHONPATH=engine/tools python3 -m packbuilder passages --lang ur .   # pack/passages.json
python3 engine/tools/jsonify_pack.py pack # pack/*.js
./build.sh                                # index.html + sw.js
./check.sh                                # checks + stale-build guard
```

Sources download once into `.cache/`, which is gitignored. The build is
deterministic: two runs from cache give byte-identical `pack/*`,
`index.html` and `sw.js`. Stanza output is cached per sentence in
`.cache/derived/`, so only the first build is slow.

## Word-selection and level-band methodology

Candidate (lemma, POS) pairs are ranked by a blended frequency score, the
mean of the log subtitle rank and the log `wordfreq` rank. The hermitdave
list is noisy after about rank 2000, so only Arabic-script tokens count, and
rows with Latin or Devanagari letters are dropped. A pair must also have a
usable dictionary entry (Urdu, the Hindi twin, or `EXTRA_LEX`) and appear at
least twice in the tagged Tatoeba corpus (removes subtitle fragments, names
and mixed-script noise).

- **A1** (600 words): every forced item, then the highest-ranked remaining
  words.
- **A2**: the next 700 by rank.
- **B1**: the next 700 by rank.

This is a reproducible proxy for CEFR level, not an official classification.
Full funnel numbers are in the generated `REPORT.md`.

**Headword spelling.** When a spelling variant has strictly more corpus
tokens than the dictionary headword, the variant becomes the headword. Four
headwords changed: پولس to پولیس, گندہ to گندا, بھارا to بھاری and پہیہ to
پہیا. The build report lists them.

## Tatoeba text-quality pipeline (`langs/ur.py` + `bad_sentences.txt`)

Urdu Tatoeba has many non-standard spellings, handled in four layers:

1. **Normalise** (`text_norm`, applied before tagging and shipping): یے/ھے
   -> ہے; ھ standing for ہ; اپ -> آپ; کہ -> کہہ; ئو -> ؤ; fused futures
   (کرونگا -> کروں گا); a word-final ئ; اے and ہوے; the لیئے/دیجیئے
   spellings; Latin punctuation (`. ? , ;`) -> `۔ ؟ ، ؛`; zero-width and bidi
   characters removed. Rewrote 406 of 2,434 linked Tatoeba sentences.
2. **Drop by pattern** (`TEXT_DROP_RE`): a first-person future without ں
   (جاؤ گا); broken agreement (ے گے, تے ہے, تیں ہیں); Latin letters or
   digits; Arabic-block letters Urdu does not use; a year used as a century
   (۱۸۸۱ صدی); کے متعلق میں; گم گیا. Dropped 86 sentences.
3. **Hand review** (`tools/bad_sentences.txt`): every Tatoeba sentence that
   reaches the shipped set is read by hand (by Claude, not a native
   speaker); ungrammatical or non-standard ones go on the list.
4. **Deduplicate**: two rows whose text is the same after `text_norm` ship
   once, with the first row's English (`translation_mismatch`).

Words that lost sentences to this pipeline were backfilled with written
ones (~150 in this pass), so every word keeps at least two sentences.

## Urdu rules in `langs/ur.py` (summary; details and edge cases in the file)

- **Tagging.** Stanza 1.14.0 with the ur default model (UD Urdu-UDTB) tags
  the corpus. `fix_token` re-reads tokens whose lemma is no dictionary word.
  Verbs are taught as the -نا infinitive, nouns as the direct singular and
  adjectives as the masculine -ا form.
- **Hindi twin.** The kaikki Urdu extract misses core words, so an entry
  from the kaikki Hindi extract fills the gap when its form is tagged with
  the Urdu spelling. That adds 4,888 lexicon entries. 243 pack words take
  their gloss from the Hindi twin. 85 pack words have a hand-written
  dictionary entry (`EXTRA_LEX`) because neither extract has a usable one.
- **Compounds.** Compound postpositions (کے لیے, کے بعد, کے بارے میں; 24 in
  the pack) are single words. So are light verbs (کام کرنا, انتظار کرنا, یاد
  آنا) and stem + vector verbs (کر دینا, چلا جانا), with 117 in the pack from
  a hand list of 136. Auxiliaries link nothing on purpose: the progressive
  رہا/رہی/رہے after a stem, a vector or passive جا/گیا/گئی/دیا outside a
  listed compound, conjunctive کر/کے, and the future گا/گی/گے. Passages do
  not count them. سکنا and چکنا do link.
- **Context rules.** A noun-looking token that closes a clause with no
  finite verb is that verb (بہت اچھا کھیلا). After a genitive or an
  adjective, a verb-looking token is a noun (نئے جوتے, مرغی کا سالن). The
  English decides a few homographs (ہوا "wind", سونا "gold", چینی "sugar").
- **More context rules** (each is a distinct disambiguation class in
  `ur.py`): imperative دو is دینا; بس means "bus" only near a postposition
  or a stop/driver word; a causative stem before the progressive, a modal
  or a vector is the -انا verb (پکا رہی); feminine kinship nouns stay
  themselves (نانی); name spellings never link (ہوائی Hawaii, آنا Anna,
  since Urdu has no capitals); a verb reading after a quantifier or
  possessive is the noun (کتنے بچے); a modal form with no verb stem before
  it is the noun (مٹھی میں سکے); the second part of a بے compound links
  nothing; a -وں verb stem at a clause end is the subjunctive (پہنچوں); a
  predicate noun right after its subject pronoun is the adjective when one
  exists (یہ مشکل ہے); آپس "each other" never folds to آپ (`NEVER_FOLD`); a
  multi-word place name is one name, never its last part (X آباد, X پور, X
  گڑھ, X نگر, نیو X, when the English has the name); an English loan spelled
  like a verb stem (لیٹ "late", پاس "pass", ہٹ "hit") is the loan when the
  English has the word — two-word loans (فٹ بال, ٹیلی ویژن, بیس بال, پلیٹ
  فارم, سوپر مارکیٹ) link nothing, and سیٹ meaning "set" is not "seat"; a
  perfect participle before a copula, ہوا or the clause end is the
  intransitive (سنا ہے is سننا, اترا is اترنا), never the causative, as is
  X-ا + رہا with no auxiliary after it (پھنسا رہا); کی read as کرنا after a
  noun with no کرنا light verb, right before a verb, is the genitive (زور
  کی لگی) — after کس/جس/کن/جن it is always the genitive (کس کی); a
  light-verb noun + کی/کیا after an ergative نے, with no later finite verb,
  is the light verb (کوشش کی is کوشش کرنا), never the genitive; an
  infinitive before ہے/تھا/پڑ-/چاہیے with a dative subject is the verb
  (توجہ دینی ہے), never a same-spelled adjective (دینی "religious") — a noun
  headword or a word after a modifier stays the noun (ٹھنڈا پانی چاہیے);
  neither half of the echo pair ملتا جلتا links.
- **Collapsed POS.** The builder keeps one entry per lemma unless a second
  POS holds 20% of its tokens. A token of the dropped POS still links the
  kept entry when both dictionary senses share a content word, or when the
  lemma is a correlative (اتنا ایسا جیسا کتنا), a deictic adverb (ادھر) or a
  side word (بائیں). گانا is one verb entry glossed "to sing; song", and the
  noun reading links it (`MERGED_POS`).
- **Forced A1 closed sets:** days, Gregorian months, seasons, numbers 0-20
  plus the tens, سو and ہزار, colours, greetings and politeness, pronouns
  with their تو/تم/آپ register, possessives, question words, core
  postpositions, conjunctions, particles, the copula, and ہونا/سکنا. The A1
  core list in `tools/forced_a1.txt` is forced too.
- **Sentences.** 3-14 tokens, B1 needs at least 5. Proper nouns are never
  linked.
- **Content policy.** The shared sensitive filter applies, with Urdu terms
  added in `ur.py`. The A1/A2 tier covers sexual content, violence, weapons,
  death, drugs and alcohol (43 sentences held at B1). Dropped at every
  level: rape and abuse, suicide and self-harm, and religious, communal or
  political side-taking (sectarian and proselytising terms, India-versus-
  Pakistan framing). Neutral vocabulary stays: مذہب, مذہبی, دینی, جمہوریت,
  جمہوری, قیامت and پادری are pack words. The hand drop list removes
  sentences that rank religions or make political claims.
- **Word-level gloss ceiling** (engine `lower_level_gloss_re`, since engine
  ef44c6e): a word whose gloss names violence, sex, drugs or alcohol cannot sit
  below B1. شراب "alcohol" and زہر "poison" moved A2 to B1; مارنا is glossed
  "to hit, to beat" (the "to kill" sense dropped) and returns to A1. The fixed
  level quotas pushed سیاست A1 to A2 and pulled احاطہ B1 to A2.

## Reading passages (`passages_src.json`, `langs/ur.py`)

The passages build needs `.cache/kaikki_ur.jsonl`, which `tools/build_pack.py`
unpacks, so run the pack build first. The texts were written by a separate
authoring pass against the final word list; per-passage notes are in
`tools/REPORT_passages.md`. The builder links word ids the same way as for
example sentences. Auxiliaries, vector verbs after a stem, the progressive
رہنا and conjunctive کر/کے are grammar and are not counted. A listed compound
that is not a pack word links its head (سن لینا to سننا, یاد کرنا to یاد).
Declared names stay names wherever they stand, since Urdu has no capitals.

Self-check stats: `--check` reports 0 coverage errors. Multiple-choice keys
found verbatim in the passage text are A1 9/44, A2 8/46, B1 13/60 (limit 15
per level). The correct option is the unique longest in A1 4/44, A2 6/46, B1
5/60 (limit 25%).
