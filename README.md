# Urdu A1-B1 vocab pack

Live at **https://bannerless-studio.github.io/urdu/**. A free,
offline-capable vocabulary trainer for Urdu, built on the shared
[`vocab-engine`](https://github.com/Bannerless-Studio/vocab-engine). It
teaches 2000 words spanning A1-B1, each with a short English gloss and a
romanisation, and at least two example sentences with English translations.
The Read tab adds 60 short reading passages with comprehension questions.

**Scope note:** this app is a vocabulary base for B1; an exam also needs
grammar, writing and speaking practice. The app teaches words, glosses,
example sentences and short reading texts only.

## Using the trainer

- **Script primer.** A "حروفِ تہجی" stage runs before A1 and teaches the
  Urdu alphabet: 40 units in 8 sets, including ٹ ڈ ڑ, nūn ghunna ں,
  do-chashmī he ھ and the hamza seat. Rule cards cover the lam-alif
  ligature لا, ؤ (hamza on vao, as in جاؤ), khaṛī zabar ٰ (اعلیٰ aʿlā), and
  ژ. It has symbol-to-sound, recognition and word-reading items. You can
  skip it with "I can read it" and bring it back later from Progress.
  macOS and most desktop browsers ship no Urdu voice, so the primer is
  text-only.
- **Today** runs one daily session: review, learn new words, listen, recall,
  sentences, then (once unlocked) a reading passage. Each stage skips itself
  when it has too little due material.
- **Words** is a searchable browser with per-set drills; **Test** has a
  placement test plus free tests; **Progress** shows stats and lets you
  export, import or reset your progress.
- **Read tab passages.** 60 short reading texts (20 each at A1, A2, B1),
  4-5 comprehension questions each. A passage's spaced re-read (after 7
  days) becomes a listening pass when every sentence has audio: the text
  stays hidden and some questions are audio-only.
- **Offline.** The page is cached on first visit and keeps working without
  a connection; a new build updates the cache in the background.
- Your progress is stored only in your browser (`localStorage`). Use
  Progress to export a backup or move it to another device.

## Script and display

- Urdu is right to left. The font is
  [Noto Nastaliq Urdu](https://fonts.google.com/noto/specimen/Noto+Nastaliq+Urdu)
  (OFL, weights 400 and 700), loaded from Google Fonts, with a taller line
  height since Nastaliq needs more vertical room. English glosses and
  romanisation stay left to right.
- Displayed text uses the Urdu codepoints (ی ک ہ ے, ۓ written ئے); matching
  also folds harakat, tatweel, ZWNJ, ۂ/ؤ/أ, and one-word spelling variants
  (لئے = لیے, چاہئے = چاہیے, تمھیں = تمہیں).
- **Romanisation** (`pron`) follows a single scheme built from Wiktionary's
  Urdu reading (falling back to the Hindi twin's, then a compound rule):
  long vowels ā ī ū e o ai au, short vowels a i u; only the retroflex
  consonants carry dots (ṭ ḍ ṛ); ش is ś, خ is x, غ is ġ, چ is c, ژ is ž, ع is
  ʿ (always written); hamza after a consonant is ʾ; a tilde marks nasals
  (mẽ, maĩ); izafat is -e. All 2000 words have a romanisation.
- Typed answers accept a fold of harakat, tatweel and ZWNJ (not part of
  ordinary Urdu writing) and of ھ (do-chashmi heh) against ہ, at every
  level; folds that would collide with another pack word (پھر/پہر,
  کھلانا/کہلانا) are rejected both ways.

## Data quality

Hand-checked samples: a 60-word stratified sample (seed 41) had 59/60
correct primary senses and 60/60 correct parts of speech (the one weak
gloss was fixed). Three sentence-link samples (90, 120 and 60 sentences)
found and fixed several disambiguation rule classes (homographs the
English decides, name-vs-word spellings, light-verb and vector-verb
splits); the current shipped sentences carry no known wrong link from
those samples.

Tatoeba has only about 2,400 Urdu sentences with an English link, so
**2,277 of the 3,023 example sentences were written for this pack** — a
high share (75%). 1,130 words have only written sentences. Written
sentences are standard Urdu of 4-9 tokens using only pack words, marked
`"src": "gen"` in `pack/sentences.json`; they are machine-written and
reviewed, but not by a native Urdu speaker.

Urdu Tatoeba text has many non-standard spellings; every Tatoeba sentence
that reaches the shipped set has been normalised, filtered and hand
reviewed for grammaticality (details in `tools/README.md`). Tatoeba has
only two permissively licensed Urdu recordings, so the pack ships no
sentence audio; speech uses the browser's ur-PK voice when one exists.

**Noun gender.** Urdu agreement depends on gender, so nouns show it in the
gloss: (m), (f), or (m/f) for common-gender person nouns (دوست, ڈاکٹر). 1,139
of 1,142 nouns are marked. A noun glossed both as a language and as a
person (جرمن, فرانسیسی) shows no gender.

The Read tab passages were machine-written by Claude and checked by
automated QA; they have not had a native-speaker review.

Level bands (A1/A2/B1) are a reproducible frequency-based proxy for CEFR,
not an official classification. No graded Urdu word list is used or
shipped. Known residuals are tracked in `TODO.md`.

## Sources and licences

| Data | Source | Licence | Used for |
|---|---|---|---|
| Spoken/subtitle frequency | [hermitdave/FrequencyWords](https://github.com/hermitdave/FrequencyWords) (`ur_full.txt`, OpenSubtitles 2018; only Arabic-script tokens kept) | CC BY-SA 4.0 | word ranking |
| Written/general frequency | [`wordfreq`](https://github.com/rspeer/wordfreq) Python package (ur) | CC BY-SA 4.0 | word ranking |
| Glosses, POS, romanisation, inflection tables | [kaikki.org](https://kaikki.org) Urdu Wiktionary extract, plus the Hindi extract for entries it links to an Urdu spelling | CC BY-SA 3.0 / GFDL | glosses, POS, `pron`, forms |
| POS tagging / lemmatisation (build time only) | [Stanza](https://stanfordnlp.github.io/stanza/) 1.14.0 (Apache-2.0), ur default model trained on UD Urdu-UDTB | model data CC BY-NC-SA 4.0 | corpus POS and lemmas, sentence links. No model files ship. |
| Example sentences | [Tatoeba](https://tatoeba.org) `urd_sentences_detailed.tsv` | CC BY 2.0 FR | sentence text (contributors in `pack/attribution.json`) |
| Sentence translations | Tatoeba `eng_sentences.tsv` + `urd-eng_links.tsv` | CC BY 2.0 FR | English translations |
| Written sentences | `tools/generated_sentences.tsv`, written for this pack | same as this repo | 2,277 sentences marked `"src": "gen"` |
| Font | Noto Nastaliq Urdu via Google Fonts | SIL OFL 1.1 | display only |

Licence: code MIT, pack data CC BY-SA 4.0, see LICENSE.

## Rebuild and publish

This repo holds the Urdu data pack and the data files its build reads.
`vocab-engine` (above) is a git submodule at `engine/` and holds the shared
UI, drill logic and pack builder; all Urdu-specific rules live in
`engine/tools/packbuilder/langs/ur.py`. For the exact rebuild/check commands
and file-by-file notes on `tools/`, see `tools/README.md` and `CLAUDE.md`.
