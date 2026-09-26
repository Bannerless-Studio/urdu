# Urdu A1-B1 vocab pack

Static data pack for a language-agnostic vocab trainer (`key: "ur"`). It has
2000 words spanning A1-B1. Each word has a short English gloss and a
romanisation, and at least two example sentences with English translations.
The Read tab adds 60 short reading passages with comprehension questions
(see "Reading passages" below).

**Live:** https://bannerless-studio.github.io/urdu/

**Script primer.** A "حروفِ تہجی" stage runs before A1 and teaches the Urdu
alphabet: 40 units in 8 sets, including ٹ ڈ ڑ, nūn ghunna ں, do-chashmī he ھ
and the hamza seat. Rule cards cover the lam-alif ligature لا, ؤ (hamza on
vao, as in جاؤ), khaṛī zabar ٰ (اعلیٰ aʿlā), and ژ, whose examples sit in the
unit note because no pack word uses it. It has symbol-to-sound, recognition
and word-reading items. Learners can skip it with "I can read it" and bring it back later from
Progress. macOS and most desktop browsers ship no Urdu voice, so the primer is
text-only (`tts: false`).

This repo holds the Urdu data pack and the Urdu data files its build reads.
[`vocab-engine`](https://github.com/Bannerless-Studio/vocab-engine) is a git submodule
at `engine/`. The engine holds the shared UI, the drill logic and the shared
pack builder, `engine/tools/packbuilder`. The builder's Urdu rules live in
`engine/tools/packbuilder/langs/ur.py`.

**Scope note:** this app is a vocabulary base for B1; an exam also needs
grammar, writing and speaking practice. The app teaches words, glosses,
example sentences and short reading texts only.

**Data quality:** a 60-word stratified sample (seed 41) has 59/60 correct
primary senses and 60/60 correct parts of speech. The one weak gloss
(انتہائی "extremely") was fixed. A hand-checked 90-sentence link sample
(seed 42) had 6 wrong links of 489 (98.8%). Four of them were rule classes
that are now fixed: homographs the English decides (ہوا "wind", سونا
"gold", چینی "sugar"), the genitive predicate (لکڑی کی ہے) and the
چاہئیے spelling. The other two came from Tatoeba typos. A second QA pass
(seed 72, 120 sentences, 689 links) found 12 wrong links in 11 sentences.
Each was a rule class, and all are now fixed. They were: a name spelled like
a word (آنا "Anna"), a plural noun read as a verb (کتنے بچے), a predicate
adjective read as its noun (یہ مشکل ہے), the second part of a بے compound,
a -وں subjunctive read as a noun, a vector verb split from its stem (رک گئ),
and senses the gloss lacked (جرمن the language, پتے "leaves"). Of the 120
sentences, 107 still ship, with 601 links and no known wrong link. The other
13 were removed by the text-quality pass below or lost their slot. A third
pass (seed 81) found 4 wrong links in a 60-item sample. Their classes are
fixed as rules: آپس read as آپ, a feminine kin noun folded to the masculine,
a multi-word place name linked by its last word, an English loan read as a
verb, a perfect participle read as the causative, and a light-verb کی read
as the genitive. See the context rules below.

Tatoeba has only about 2,400 Urdu sentences with an English link, so **2,277
of the 3,023 example sentences were written for this pack**. That is a high
share (75%), and 1,130 words have only written sentences. The written
sentences are marked `"src": "gen"` in `pack/sentences.json` and listed in
`tools/generated_sentences.tsv`. The other 746 come from Tatoeba. Written
sentences are standard Urdu of 4-9 tokens that use only pack words. They
went through the same tagger and linker as Tatoeba. They are machine-written
and reviewed, but not by a native Urdu speaker. Tatoeba has only two
permissively licensed Urdu recordings, so the pack ships no sentence audio,
and speech uses the browser's ur-PK voice when one exists. Every word has a
romanisation. Known residuals are in `TODO.md`.

**Tatoeba text quality.** Urdu Tatoeba has many non-standard spellings. Three
layers handle them, all in `langs/ur.py` except the hand list:

1. **Normalise.** `text_norm` rewrites the text before tagging and shipping.
   It fixes these classes:
   - یے or ھے for ہے;
   - ھ standing for ہ;
   - اپ for آپ;
   - کہ for کہہ;
   - ئو for ؤ;
   - fused futures (کرونگا becomes کروں گا);
   - a word-final ئ;
   - اے and ہوے;
   - the لیئے/دیجیئے spellings;
   - Latin punctuation (. ? , ;) becomes ۔ ؟ ، ؛;
   - zero-width and bidi characters are removed.

   It rewrote 406 of the 2,434 linked Tatoeba sentences.
2. **Drop by pattern.** `TEXT_DROP_RE` drops these classes:
   - a first-person future without ں (جاؤ گا);
   - broken agreement (ے گے, تے ہے, تیں ہیں);
   - Latin letters or digits;
   - Arabic-block letters Urdu does not use;
   - a year used as a century (۱۸۸۱ صدی);
   - کے متعلق میں;
   - گم گیا.

   It dropped 86 sentences.
3. **Hand review.** Every Tatoeba sentence that reaches the shipped set was
   read by hand. The reader was Claude, not a native speaker. Ungrammatical
   or non-standard ones are listed in `tools/bad_sentences.txt`, 181 so far.
   The list is keyed by normalised text, not by Tatoeba id, and a rebuild
   drops them. When a new Tatoeba sentence enters the shipped set after a
   rebuild, read it and add it to the list if it is bad.
4. **Deduplicate.** Two rows whose text is the same after `text_norm`
   ship once, with the first row's English (`translation_mismatch`). A
   written sentence's English never carries a gloss-style "(...)" tag; the
   loader strips one if it appears.

Words that lost sentences were backfilled with written ones. About 150 were
written in this pass, so every word keeps at least two sentences.

**Noun gender.** Urdu agreement depends on gender, so nouns show it in the
gloss: (m), (f), or (m/f) for common-gender person nouns such as دوست and
ڈاکٹر. The Urdu kaikki headword and its Hindi twin supply the gender. When
they disagree or both are missing, the noun is unmarked unless
`tools/gender_overrides.txt` sets it. That file has 124 lines, and a "-"
line leaves a noun unmarked. 1,139 of 1,142 nouns are marked. Closed sets
are pinned in `ur.py` before the dictionary guess: every Gregorian month and
day is masculine except جمعرات, and seasons are feminine. A noun glossed
both as a language and as a person (جرمن, فرانسیسی) shows no gender. Only
nouns carry a mark: the build strips (m)/(f) from every other POS and
reports the count as `gender_marks_stripped_from_non_nouns`. The shared
possessive-gender link check runs only for nouns with one gender.

**Headword spelling.** When a spelling variant has strictly more corpus
tokens than the dictionary headword, the variant becomes the headword.
Four headwords changed: پولس to پولیس, گندہ to گندا, بھارا to بھاری and
پہیہ to پہیا. The build report lists them.

## Reading passages (Read tab)

`tools/passages_src.json` holds 60 short reading texts, 20 each at A1, A2
and B1, with 4-5 comprehension questions each. It follows the rules block of
`persian/tools/passages_src.json`. The builder turns it into
`pack/passages.json`:

```
PYTHONPATH=engine/tools python3 -m packbuilder passages --lang ur .   # --check: report only
python3 engine/tools/jsonify_pack.py pack                             # passages go into passages.js
```

The passages build needs `.cache/kaikki_ur.jsonl`, which `tools/build_pack.py`
unpacks, so run the pack build first. The texts were written by a separate
authoring pass against the final word list; per-passage notes are in
`tools/REPORT_passages.md`. The builder links word ids the same way
as for example sentences. Auxiliaries, vector verbs after a stem, the
progressive رہنا and conjunctive کر/کے are grammar and are not counted. A
listed compound that is not a pack word links its head (سن لینا to سننا,
یاد کرنا to یاد). Declared names stay names wherever they stand, since Urdu
has no capitals to mark them.

Self-check stats:
- **Coverage:** `--check` reports 0 errors, and every counted token in the 60
  passages links a pack word or a declared name or out-of-pack word.
- **Verbatim keys:** multiple-choice keys found verbatim in the passage text
  are A1 9/44, A2 8/46 and B1 13/60. The limit is 15 per level.
- **Unique longest:** the correct option is the unique longest in A1 4/44,
  A2 6/46 and B1 5/60. The limit is 25%.

The passages and questions are machine-written by Claude and checked by the
automated QA pass. They have not had a native-speaker review.

## Script and display

- **Direction and font.** Urdu is right to left. `pack/pack.json` sets `rtl:
  true`, `langTag: "ur"` and a 2.6 line height, because Nastaliq needs about
  2.4-3. The font is
  [Noto Nastaliq Urdu](https://fonts.google.com/noto/specimen/Noto+Nastaliq+Urdu)
  (OFL, weights 400 and 700), loaded from Google Fonts. English glosses and
  romanisation stay left to right.
- **Normalisation.** Displayed text uses the Urdu codepoints: ی ک ہ ے, with
  ۓ written ئے. Arabic ي ك ه ة are converted at build time. Matching also
  strips harakat, tatweel and ZWNJ, and folds ۂ ؤ أ. One-word spelling
  variants are unified: لئے = لیے, چاہئے = چاہیے, تمھیں = تمہیں.
- **Romanisation** (`pron`) comes from Wiktionary's Urdu reading. It falls
  back to the Hindi twin's reading, then to a rule for compounds: a verb stem
  from the infinitive, or the -ی/-ے form of an -ا word (بھیج دینا = bhej
  denā, کے بارے میں = ke bāre mẽ). `pron_norm` in `ur.py` maps every source
  onto one scheme:
  - Long vowels are ā ī ū e o ai au, and short vowels are a i u.
  - Only the retroflex consonants carry dots: ṭ ḍ ṛ.
  - ش is ś, خ is x, غ is ġ, چ is c and ژ is ž. ع is ʿ and is always written.
  - Hamza after a consonant is ʾ (jurʾat, masʾala). It is the only hamza
    glyph; no ASCII apostrophe appears. A tilde marks nasals (mẽ, maĩ).
  - A word ending in consonant + ی ends in -ī (axlāqī, kārkardagī).
  - There is one pron word per written word (zyāda tar, zimme dārī).
  - Word-final ہ after a consonant is -a (kamra). After a long vowel, in a
    monosyllable, and in جگہ and سالگرہ it is -h (rāh, dah, jagah).
  - Izafat is -e (barāh-e karam).
  - The spelling decides retroflex and aspirate marks, and an ā that has no
    alif (toṛnā, baṛhnā, xarābī).

  A small per-token table fixes source errors inside compounds too (gunā,
  baʿd, dah).
- **Drills.** Typed production is on: `typing: {caseSensitive: false,
  accents: lenient, strictFromLevel: null}`. Lenient accents fold harakat,
  tatweel and ZWNJ, none of which ordinary Urdu writing includes, and, as of
  engine `122d88a`, also fold ھ (do-chashmi heh) against ہ, at every level.
  Every fold is guarded against collisions with another pack word: پھر/پہر
  and کھلانا/کہلانا are rejected both ways because both spellings are pack
  words (PACK_SCHEMA.md).

## Urdu rules (summary; details in `langs/ur.py`)

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
  a hand list of 136. Auxiliaries are grammar and link nothing, on purpose:
  the progressive رہا/رہی/رہے after a stem, a vector or passive جا/گیا/گئی/دیا
  outside a listed compound, conjunctive کر/کے, and the future گا/گی/گے.
  Passages do not count them. سکنا and چکنا do link.
- **Context rules.** A noun-looking token that closes a clause with no
  finite verb is that verb (بہت اچھا کھیلا). After a genitive or an
  adjective, a verb-looking token is a noun (نئے جوتے, مرغی کا سالن). The
  English decides a few homographs.
- **More context rules.**
  - Imperative دو is دینا.
  - بس means "bus" only near a postposition or a stop or driver word.
  - A causative stem before the progressive, a modal or a vector is the -انا
    verb (پکا رہی).
  - Feminine kinship nouns stay themselves (نانی).
  - Name spellings are never linked (ہوائی Hawaii, آنا Anna). Urdu has no
    capitals, so sentence-initial names get no surface fallback.
  - A verb reading after a quantifier or possessive is the noun (کتنے بچے).
  - A modal form with no verb stem before it is the noun (مٹھی میں سکے).
  - The second part of a بے compound links nothing.
  - A -وں verb stem at a clause end is the subjunctive (پہنچوں).
  - A predicate noun right after its subject pronoun is the adjective when
    one exists (یہ مشکل ہے).
  - آپس "each other" never folds to آپ (`NEVER_FOLD`).
  - A multi-word place name is one name, never its last part: X آباد, X پور,
    X گڑھ, X نگر and نیو X are names when the English has the name.
  - An English loan spelled like a verb stem (لیٹ "late", پاس "pass", ہٹ
    "hit") is the loan when the English has the word. Two-word loans (فٹ بال,
    ٹیلی ویژن, بیس بال, پلیٹ فارم, سوپر مارکیٹ) link nothing, and سیٹ meaning
    "set" is not "seat".
  - A perfect participle before a copula, ہوا or the clause end is the
    intransitive (سنا ہے is سننا, اترا is اترنا), never the causative. So is
    X-ا + رہا with no auxiliary after it (پھنسا رہا "remained stuck").
  - کی read as کرنا after a noun with no کرنا light verb, right before a
    verb, is the genitive (زور کی لگی). After کس/جس/کن/جن it is always the
    genitive (کس کی "whose").
  - A light-verb noun + کی/کیا after an ergative نے, with no later finite
    verb, is the light verb (کوشش کی is کوشش کرنا), never the genitive.
  - An infinitive before ہے/تھا/پڑ-/چاہیے with a dative subject is the verb
    (توجہ دینی ہے), never a same-spelled adjective (دینی "religious"). A noun
    headword or a word after a modifier stays the noun (ٹھنڈا پانی چاہیے).
  - Neither half of the echo pair ملتا جلتا links (ملتے جلتے "alike").
- **Collapsed POS.** The builder keeps one entry per lemma unless a second
  POS holds 20% of its tokens. A token of the dropped POS still links the
  kept entry when both dictionary senses share a content word, or when the
  lemma is a correlative (اتنا ایسا جیسا کتنا), a deictic adverb (ادھر) or a
  side word (بائیں). گانا is one verb entry glossed "to sing; song", and the
  noun reading links it (`MERGED_POS`).
- **Forced A1 closed sets.** These are days, Gregorian months, seasons,
  numbers 0-20 plus the tens, سو and ہزار, colours, greetings and
  politeness, pronouns with their تو/تم/آپ register, possessives, question
  words, core postpositions, conjunctions, particles, the copula, and
  ہونا/سکنا. The A1 core list in `tools/forced_a1.txt` is forced too.
- **Sentences.** Sentences have 3-14 tokens, and B1 needs at least 5.
  Proper nouns are never linked.
- **Content policy.** The shared sensitive filter applies, with Urdu terms
  added in `ur.py`. The A1/A2 tier covers sexual content, violence, weapons,
  death, drugs and alcohol, and 43 such sentences are held at B1. These are
  dropped at every level: rape and abuse, suicide and self-harm, and
  religious, communal or political side-taking (sectarian and proselytising
  terms, and India-versus-Pakistan framing). Neutral vocabulary stays: مذہب,
  مذہبی, دینی, جمہوریت, جمہوری, قیامت and پادری are pack words. The hand
  drop list removes sentences that rank religions or make political claims.
  One gloss sense was skipped by the A1/A2 gloss scan.

## Sources and licences

| Data | Source | Licence | Used for |
|---|---|---|---|
| Spoken/subtitle frequency | [hermitdave/FrequencyWords](https://github.com/hermitdave/FrequencyWords) (`ur_full.txt`, OpenSubtitles 2018; only Arabic-script tokens kept) | CC BY-SA 4.0 | word ranking |
| Written/general frequency | [`wordfreq`](https://github.com/rspeer/wordfreq) Python package (ur) | CC BY-SA 4.0 | word ranking |
| Glosses, POS, romanisation, inflection tables | [kaikki.org](https://kaikki.org) Urdu Wiktionary extract, plus the Hindi extract for entries it links to an Urdu spelling | CC BY-SA 3.0 / GFDL | glosses, POS, `pron`, forms |
| POS tagging / lemmatisation (build time only) | [Stanza](https://stanfordnlp.github.io/stanza/) 1.14.0 (Apache-2.0), ur default model trained on UD Urdu-UDTB | model data CC BY-NC-SA 4.0 | corpus POS and lemmas, sentence links. No model files ship. |
| Example sentences | [Tatoeba](https://tatoeba.org) `urd_sentences_detailed.tsv` | CC BY 2.0 FR | sentence text (contributors in `pack/attribution.json`) |
| Sentence translations | Tatoeba `eng_sentences.tsv` + `urd-eng_links.tsv` | CC BY 2.0 FR | English translations |
| Written sentences | `tools/generated_sentences.tsv`, written for this pack | same as this repo | 2,144 sentences marked `"src": "gen"` |
| Font | Noto Nastaliq Urdu via Google Fonts | SIL OFL 1.1 | display only |

Licence: code MIT, pack data CC BY-SA 4.0, see LICENSE.

No graded Urdu word list is used or shipped.

## Level bands

Candidate (lemma, POS) pairs are ranked by a blended frequency score. The
score is the mean of the log subtitle rank and the log `wordfreq` rank. The
hermitdave list is noisy after about rank 2000, so only Arabic-script tokens
count, and rows with Latin or Devanagari letters are dropped. A pair must
also have a usable dictionary entry (Urdu, or the Hindi twin, or the hand
list). It must appear at least twice in the tagged Tatoeba corpus. That
removes subtitle fragments, names and mixed-script noise.

- **A1** (600 words): every forced item, then the highest-ranked remaining
  words.
- **A2**: the next 700 by rank.
- **B1**: the next 700 by rank.

This is a reproducible proxy for CEFR level. It is not an official CEFR
classification.

## Layout

```
pack/                pack.json, words.json, sentences.json, script.json, attribution.json (+ generated .js)
engine/              git submodule -> vocab-engine (UI, drills, tools/packbuilder, langs/ur.py)
tools/
  build_pack.py      shim: python3 -m packbuilder build --lang ur --repo .
  gloss_overrides.json   hand gloss fixes ("lemma|pos", folded keys; covers the top 300)
  bad_sentences.txt  hand-reviewed Tatoeba drops, keyed by normalised text
  gender_overrides.txt   noun gender where the Urdu and Hindi dictionaries disagree or lack it
  forced_a1.txt      A1 core list (closed sets are in langs/ur.py)
  generated_sentences.tsv  sentences written for this pack (append only: line order sets ids)
  passages_src.json  reading passages + questions (source of pack/passages.json)
  id_map_v1.json     frozen "lemma|pos" -> word id (keeps learner progress across rebuilds)
  requirements.txt   packbuilder deps + stanza
  REPORT.md          generated build report
build.sh             builds index.html + sw.js from pack/ + engine/
check.sh             packbuilder check + engine validator + stale-build guard
```

## Rebuilding

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
`.cache/derived/`, so only the first build is slow. To build against a
vocab-engine checkout other than the submodule, set
`PACKBUILDER_PATH=../vocab-engine/tools` for `tools/build_pack.py` and
`./check.sh`. QA helpers run with
`PYTHONPATH=engine/tools python3 -m packbuilder {scan,sample} --lang ur --repo .`.
