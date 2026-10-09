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
- Republish 09e90bc: sentence spans (16830/16897 linked words placed); inflected forms now cloze targets.
- Republish ef44c6e: شراب, زہر A2→B1; مارنا "to hit, to beat" B1→A1 (quota: سیاست A1→A2, احاطہ B1→A2); deleted override keys آباد|adj, اترانا|verb, بائیں|adj, جلدی|adj, عدم|noun, عربی|noun, پھنسانا|verb, پیدل|adj; set-counter and no-voice planner fixes

## Typed production
- RESOLVED (engine `122d88a`). `typing.accents: lenient` now folds ھ
  (do-chashmi heh, the aspiration marker in بھ/ٹھ/تھ/دھ etc.) against ہ
  (heh), alongside harakat, tatweel and ZWNJ, at every level (PACK_SCHEMA.md
  "Lenient typing letter folds"). Every lenient fold is guarded: a typed
  answer that matches only after folding is rejected when it spells another
  pack word. Urdu has 2 colliding pairs from this fold (PACK_SCHEMA.md's
  collision table): پھر (w0067, A1) also folds to پہر (w1964, B1), and
  کھلانا (w2016, A2) also folds to کہلانا (w1743, B1); typing one for the
  other is rejected both ways.

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
- گانا is now two entries, noun "song" and verb "to sing" (v1.1, 2026-09-26):
  `core/words.py` gained `LanguageSpec.second_entry_overlap_exempt`, a per-language
  hand list of lemmas whose second POS entry is admitted despite weak automatic
  evidence (a low dictionary-sense score or high translation overlap with the first
  entry: "to sing a song" keeps "song" in most translations of "to sing", and a
  forced word's own sense often scores 0 for lack of bag evidence) once
  `SECOND_ENTRY_SHARE` already shows real, independent corpus support for both POS
  (گانا was 50/50). A true same-sense signal (shared gloss stem, same POS group, a
  nominalised adjective) is never exempted. `ur.py`'s old `MERGED_POS` link-only
  workaround is gone; `گانا` is the exempt lemma.
- Kin closed set audited (v1.1, 2026-09-26): ماں باپ بھائی بہن بیٹا بیٹی دادا دادی
  نانی چچا چچی شوہر بیوی were already in the pack; نانا was restored (forced A1,
  matching نانی). ماموں ممانی خالہ خالو پھوپھی remain absent: the Hindi twin has
  none of them either except مामा (Hindi A2), so there is no twin level to force
  them at, and the forced mechanism only ever places a word at A1 (`assign_levels`:
  every forced key goes to the first band). Restoring them would need either a
  forced-level mechanism beyond A1, or enough natural corpus frequency to rank into
  the pool on their own.
- Passage text ships verbatim (`passages.py` has no text-normalising hook). After a
  headword respelling, grep `tools/passages_src.json` for the old spelling (done for
  پولس and گندہ on 2026-09-26).
- At the 2,000 cap, restoring نانا and the گانا noun entry (v1.1, 2026-09-26) pushed
  out رومال and پنیر (both B1, neither closed-set). کمبل, گلاس and ورزش were already
  out before this round and stay out. `keep_keys` cannot bring back a word ranked
  outside the candidate pool, because it gets no record (core); only `forced` (A1
  only) can.
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

## Live check residuals (2026-09-26, 8/9 PASS on f4dd858)
- Engine: word MC options show the option number flush against the Urdu word under RTL
  (`.num` margin resolves on the outer side of a dir=ltr span) — engine fix in progress
  (branch engine-rtl-spacing), republish after.
- Engine: Today plan lists "Listen N items" although no Urdu voice exists and 0 Listen
  items are served — same fix round.
- Words search does not match Urdu text inside English glosses (a query "کا/کی/کے" finds
  nothing because those are gloss fragments, not headwords). By design for now.
- ٹیک does not find ٹھیک: ھ is never folded (documented tradeoff in the engine fold).
- No lam-alif primer unit; the only لا example is والا (ligature intact under tint).

## Republish a1a290b (2026-10-08, port wave 3)
- Republish a1a290b: typed modes, day-aware scheduling, reading rotation, goals, pairs (script units keep their Review share), frequency tiers, Progress v2, redesigned tabs, session estimates. Pack diff vs b8517a8: every word gains `ft` (A1 [100,440,60] A2 [0,525,175] B1 [0,420,280] ambient/core/peripheral); pack.json gains exactly the port flag block + `eta`; sentences, passages, script, attribution identical. `eta` measured with `tests/eta_checks.js --pack pack --calibrate --sessions 400` (tools/eta.json, curve + knownCurve; out-of-sample gate seeds 8/9/10 PASS 4/4); check.sh now runs `packbuilder enrich --check`.
- Migration proof: rollback hash b8517a8f548aa78441845b97b245721cd90140cd; previous live md5 index 39a51e697bb81899ab9b87e2fd0ea88d, sw c4b377f5bf5606af8dac08800431e9d9. Storage: new fields day/sn/t/u/f/p/pm/pv/pause/read.done s,ls/today.tw on first use (script records gain t/u, never p); boot writes nothing; previous build ef44c6e/aa00571 carries them (migration [port], primer learned).
- Live proof 7a71fb0 (2026-10-08): KEEP. Live == local build (index d938928e23973728279780fee40d6b87, sw 7fbb5610af65d7e6f328f85f907c3560). 12-session seed (primer learned) byte-equal after boot/reload/Progress, no backup keys; one Today session wrote w/sets/sessions/script/sn/day/pm; the record booted on b8517a8 byte-equal, one key vocab_ur, a session ran there; back on live byte-equal; 13/13 checks, 0 errors.
- Live proof 8745de1 (2026-10-09) KEEP: rollback hash f2610a8; md5 index d938928e... -> f3af5904a535068ec3565121f047d2fa, sw 7fbb5610... -> 3d78cdac0510b40265129a38b8dfb032 (live == local). 12-session seed (primer learned) played on f2610a8 byte-equal after boot/reload/Progress open (leaving Progress adds only prog.pv); one Today session writes w/sets/sessions/script/sn/day/pm; that record boots on f2610a8 byte-equal (boot + reload), one key vocab_ur, a session runs there; back on live byte-equal; 0 console errors, 0 failed requests. Scratch 13/13 and live 13/13. Storage: new fields read.done[pid].t, prog.script choice on placement; boot writes nothing.
