# TODO (v2 candidates)

Residuals from the v1 build and QA. The rules already in place are in
`engine/tools/packbuilder/langs/ur.py` and summarised in the README.

## Engine submodule
- `engine/` is pinned to vocab-engine 3fd45bf (includes 2b1e21e): Urdu search folding, reduplicated-query folding, a
  ligature-safe tint, Escape, and RTL bidi isolation. That commit still has
  no `langs/ur.py`. Until `ur.py` is committed there and the submodule is bumped, build
  and check with `PACKBUILDER_PATH=../vocab-engine/tools`.
- `check.sh`'s stale-build guard fails until `index.html` and `sw.js` are
  committed. The repo has no commits yet.

## Passages
- The passages were authored against this word list. `--check` reports 0
  errors, and `pack/passages.json` is built.
- Passages have had automated QA only, with no native-speaker review.
- The passages build reads `.cache/kaikki_ur.jsonl`, which only the pack
  build unpacks. Run `tools/build_pack.py` first.

## Sentences
- 2,277 of 3,023 sentences were written for the pack, and 1,130 words have
  only written sentences. More Tatoeba Urdu with English links, or a second
  licensed corpus, would replace them. The written sentences have had no
  native-speaker review.
- There is no audio: Tatoeba has only 2 permissive Urdu recordings. Recorded
  audio is planned via Piper ur_PK (engine `docs/AUDIO.md`).
- Tatoeba text quality is handled in three layers: `text_norm`, `TEXT_DROP_RE`
  and the hand list `tools/bad_sentences.txt` (README). The hand review
  covers only sentences that ship. After any rebuild that changes selection,
  read the Tatoeba sentences that newly ship and drop-list the bad ones.
  The reader so far is Claude, not a native speaker.
- Link ambiguity that remains:
  - لگا is sometimes read as لگنا where لگانا is meant. The causative rule
    fixes it before رہ-, سک-, چک-, کر and vectors, but not in other positions.
  - شکر can mean sugar or thanks.
  - پتا/پتہ covers address, knowing and leaf in one gloss. Splitting it
    into separate entries needs a sense-level link.
  - A predicate noun that is also an adjective becomes the adjective only
    right after a subject pronoun or an intensifier. After a postposition
    it stays the noun, so "آنکھ سے اندھا ہے" still links the noun.
  - After a noun, ہنس کے is read as the genitive (ہنس "swan") rather than
    the conjunctive "laughing". The genitive is far more common.
  - A light verb split by its subject (ان کا خیال کون رکھے گا) links only
    the noun.
- Collapsed POS readings that still link nothing, because the dictionary
  senses differ: اور "more" (ایک اور, the pack has only "and"), سونا "gold"
  read as a noun where the pack has the verb, موثر and وسیع read as nouns.
  Two entries per lemma need either a 20% token share or a core change.
- گانا is one entry, the verb "to sing; song". A separate noun entry needs a
  core change to the one-entry-per-lemma gate (`core/words.py`).
- Passage text ships verbatim (`passages.py` has no text-normalising hook). After a
  headword respelling, grep `tools/passages_src.json` for the old spelling (done for
  پولس and گندہ on 2026-09-26).
- At the 2,000 cap, restoring the neutral religious and civic words pushed
  out نانا, رومال, کمبل, گلاس and ورزش. نانی stays. رومال came back when
  the stative پھنسا رہا stopped counting for پھنسانا, which then fell below the
  cut. Its only support was a causative پھنسا دیا. `keep_keys` cannot keep it,
  because a word ranked outside the candidate pool gets no record (core).
- چینی "Chinese" left the pack once the sugar sense stopped counting for it.
  It could come back as a second entry.

## Auxiliary links
- Auxiliaries (رہا, جا/گیا/گئی, دیا, کر, گا) link nothing, while the Hindi pack links
  رہنا, جانا and دینا. Linking them inside `post_resolve` was tried (2026-09-26): 326
  sentences gained links, but corpus counts moved. That changed 1,785 ranks,
  swapped one word (گلاس for لے جانا) and 8 sentences, and renumbered sentence
  ids. A core hook that adds links to already-selected sentences, such as
  `spec.augment_links(toks, resolved, links)` called after selection, would
  allow it without that churn. The same pass would link لے جاؤ to لے جانا, not
  لینا.

## Words and glosses
- Noun gender comes from the Urdu headword, then the Hindi twin, then
  `tools/gender_overrides.txt`. The 109 dictionary mismatches that no override
  covers are unmarked, but no pack noun is among them. When a new noun enters
  the pack, check the build report's `ur_gender` stat.
- One `pron` rule is a judgement call: final ہ is -h in جگہ and سالگرہ
  (`PRON_KEEP_H`), because the h is spoken there. A native pass over -ah
  and -a words would confirm the list.
- The pronoun میں is romanised maĩ, and the postposition میں is mẽ.
- ترکی and جرمنی are name spellings (Turkey, Germany). They are not pack
  words, and passages count them as names.
- 243 glosses come from the Hindi twin and 85 words have hand dictionary
  entries. A native pass over both would catch register slips.
- The standalone شروع "beginning" (شروع میں "at first") is not a pack word,
  only شروع کرنا and شروع ہونا are.
- The passage author asked for more words. سینما ranks below the cutoff.
  نرم, ڈبہ, کرایہ, مسافر, برتن, کلو and بجنا stay at B1, because the builder
  can force words into A1 only. An A2 force list would help. At the 2,000
  cap, اداسی "sadness" is the latest word pushed out. It went when پہنچ
  (verb forms misread as a noun) was fixed and ورزش and گلاس moved up.
- عجائب is forced so that عجائب گھر ("museum") links. A multi-word forced
  entry, or عجائب گھر as a phrase word, would be cleaner.

## Script primer and display
- `ur-zhe` (ژ) has no example word in the pack. Its note gives ٹیلی ویژن
  and ژالہ, so the validator warning "no ex" stays.
- `tts: false`: macOS lists no Urdu voice (`say -v ?` has only hi_IN). If a
  ur-PK voice becomes common, set `tts` and ear-check `script_say` (letter +
  zabar).
- Line height 2.6 for Noto Nastaliq Urdu still needs a visual check in the
  built page, to confirm descenders don't clip in cards and passages.
