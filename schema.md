# Overview

This document describes the current lesson data structures used by EPUB AI Importer.

Sources used:

- `src/lesson/buildLesson.ts`
- `src/schema/validate.ts`
- `src/pipeline/generationPipeline.ts`
- `src/schema/types.ts`
- `src/schema/normalization.ts`
- `src/schema/namedEntities.ts`
- `schemas/lesson.schema.json`
- checked-in lesson JSON fixtures under `content/` and repository validation tests

This document is intentionally strict. It reflects the current implementation, including compatibility rules, and does not introduce new fields or nested objects that are not present in code.

# Schema Stabilization Milestone

This file is the canonical schema documentation location for the current
project. Do not create a competing `docs/schema.md` unless this document is
retired deliberately.

The stabilization milestone should distinguish three things:

- **Current runtime schema**: what generated lesson JSON and validators accept
  today.
- **Planned canonical fields**: fields that are intended to become stable
  public contracts, but may not be fully enforced by runtime code yet.
- **Debug-only diagnostics**: QA metadata from debug/full-book regression,
  including Named Entity review and dry-run learning decisions. Debug metadata
  is not canonical lesson content and must not be written into lesson JSON
  unless a future schema migration explicitly adds it.

When planned schema language differs from current runtime behavior, this
document calls that out instead of silently replacing the current contract.

# Lesson Schema

Current generated lesson objects have this top-level structure:

```json
{
  "id": "string",
  "schemaVersion": 1,
  "fragmentId": "sha1:...",
  "generationHash": "sha1:...",
  "title": "string",
  "text": "string",
  "meta": {
    "bookId": "string",
    "bookTitle": "string",
    "bookLanguage": "string",
    "author": "string",
    "seriesId": "string?",
    "seriesTitle": "string?",
    "sectionIndex": 1,
    "sectionTitle": "string",
    "fragmentIndex": 1,
    "fragmentsInSection": 1,
    "sourceFile": "string?",
    "stats": {
      "wordCount": 0,
      "uniqueWordCount": 0,
      "paragraphCount": 0,
      "vocabularyCount": 0,
      "levelDistribution": {
        "A1": 0,
        "A2": 0,
        "B1": 0,
        "B2": 0,
        "C1": 0,
        "C2": 0
      }
    }
  },
  "vocabulary": [],
  "namedEntities": []
}
```

Top-level fields:

- `id: string`
- `schemaVersion: 1`
- `fragmentId: string`
- `generationHash: string`
- `title: string`
- `text: string`
- `meta: LessonMeta`
- `vocabulary: VocabularyItem[]`
- `namedEntities?: NamedEntityItem[]`

Exact generation behavior:

- `id` is generated as `{bookId}_sct{sectionIndex}_f{fragmentIndex}` by current `buildLesson`.
- `fragmentId` is `sha1:` plus the SHA-1 hash of normalized fragment text.
- `generationHash` is produced from generation config in `generationPipeline`.
- `title` is `{bookTitle} - {sectionTitle || "Section N"} (Fragment {fragmentIndex})`.
- `text` is normalized fragment text with normalized line endings, trailing whitespace removed per line, and outer whitespace trimmed.

`meta` is flat. There is no current generated structure like `meta.book`, `meta.chapter`, or `meta.fragment`.

`namedEntities` is optional and present only when spaCy NER produced lesson-level
entities. It is separate from `vocabulary`; named entities are not vocabulary
items and do not receive CEFR levels.

`meta` fields generated today:

- `bookId: string`
- `bookTitle: string`
- `bookLanguage: string`
- `author: string`
- `seriesId?: string`
- `seriesTitle?: string`
- `sectionIndex: number`
- `sectionTitle: string`
- `fragmentIndex: number`
- `fragmentsInSection: number`
- `sourceFile?: string`
- `stats: LessonStats`

`meta.stats` fields generated today:

- `wordCount: number`
- `uniqueWordCount: number`
- `paragraphCount: number`
- `vocabularyCount: number`
- `levelDistribution: Record<"A1" | "A2" | "B1" | "B2" | "C1" | "C2", number>`

Important compatibility note:

- The runtime JSON Schema and validation tests still accept a legacy lesson-meta variant using `chapterIndex`, `chapterTitle`, and `fragmentsInChapter`.
- Current generation code writes `sectionIndex`, `sectionTitle`, and `fragmentsInSection`.
- `buildLesson` still supports legacy input names internally, but current generated output is the `section*` form.
- Runtime TypeScript validation and `schemas/lesson.schema.json` both support
  optional `namedEntities`.
- Debug-only Named Entity review fields are intentionally excluded from
  canonical lesson JSON and from `schemas/lesson.schema.json`.

About the fields requested in the task:

- `meta.book.id`, `meta.book.title`, `meta.book.author` do not exist as nested fields. Current equivalents are `meta.bookId`, `meta.bookTitle`, and `meta.author`.
- `meta.chapter.index`, `meta.chapter.title`, `meta.chapter.sourceFile` do not exist as nested fields. Current equivalents are `meta.sectionIndex`, `meta.sectionTitle`, and `meta.sourceFile`.
- `meta.fragment.indexInChapter` and `meta.fragment.totalInChapter` do not exist as nested fields. Current equivalents are `meta.fragmentIndex` and `meta.fragmentsInSection`.
- `meta.stats.level` and `meta.stats.score` do not exist in the current lesson schema.

# Vocabulary Schema

Current normalized vocabulary items use this structure:

```json
{
  "word": "string",
  "lemma": "string",
  "lemmaId": "string",
  "pos": "noun | verb | adjective | adverb | phrase | phrasal_verb | prepositional_verb",
  "type": "word | phrase | phrasal_verb | prepositional_verb",
  "source": "dictionary_phrasal | spacy_phrase | fixed_expression | word",
  "wordKey": "string",
  "level": "A1 | A2 | B1 | B2 | C1 | C2",
  "levelSource": "cefr_frequency",
  "difficultyScore": 0.0,
  "difficultySource": "zipf",
  "ipa": "string",
  "translations": {
    "ru": {
      "translation": "string"
    }
  },
  "definition": "string",
  "example": "string"
}
```

For schema-stabilization work, `VocabularyItem` is the canonical name for this
object shape inside `lesson.vocabulary`.

Field list:

- `word: string` required
- `lemma: string` required
- `lemmaId: string` required
- `pos: "noun" | "verb" | "adjective" | "adverb" | "phrase" | "phrasal_verb" | "prepositional_verb"` required
- `type?: "word" | "phrase" | "phrasal_verb" | "prepositional_verb"`
- `source?: "dictionary_phrasal" | "spacy_phrase" | "fixed_expression" | "word"`
- `wordKey: string` required
- `level: "A1" | "A2" | "B1" | "B2" | "C1" | "C2"` required
- `levelSource?: "vocabulary_index" | "cefr_frequency" | "phrase_index" | "dictionary_phrase" | "head_verb_index" | "phrase_fallback" | "difficulty_heuristic"`
- `difficultyScore?: number`
- `difficultySource?: "zipf" | "cefr_frequency" | "vocabulary_index" | "difficulty_heuristic" | "none"`
- `ipa?: string`
- `translations: Record<string, { translation: string }>` required at runtime
- `definition?: string`
- `example?: string`
- `exampleTranslation?: string`
- `exampleStart?: number`
- `exampleEnd?: number`
- `matchedForm?: string`
- `phraseMatch?: { text: string; startInExample: number; endInExample: number }`

Current `matchedForm` contract:

- Runtime validation requires `matchedForm` when an ordinary single-token
  vocabulary item has `example`.
- For phrase-like vocabulary items with `example`, runtime validation accepts
  either a valid single-token `matchedForm` or a valid `phraseMatch`.
- `matchedForm` is a single-token observed surface inside `example`.
- `exampleStart` and `exampleEnd` locate the full `example` inside
  `lesson.text`; they do not currently locate a phrase match inside the
  example.

Phrase match alignment:

- `phraseMatch` is optional schema/runtime metadata for phrase-like vocabulary
  items (`phrase`, `phrasal_verb`, `prepositional_verb`, or multiword
  word/lemma values).
- `phraseMatch.text` is the observed phrase surface inside `example`.
- `phraseMatch.startInExample` and `phraseMatch.endInExample` are offsets inside
  `example`, not offsets inside `lesson.text`.
- When `exampleStart` is present, a lesson-text phrase span can be derived as
  `exampleStart + phraseMatch.startInExample` and
  `exampleStart + phraseMatch.endInExample`.
- `phraseMatch` is alignment metadata only. It does not replace
  `matchedForm`, `wordKey`, lemma/POS identity, CEFR level, translations, or
  example offsets.
- Current example extraction emits `phraseMatch` for phrase-like vocabulary
  items when the selected example contains exactly one deterministic phrase
  surface from `word` or `lemma`. It first uses case-sensitive matching, then a
  case-insensitive exact fallback that stores the observed cased substring from
  the example. It skips repeated, partial, inflected, fuzzy, and
  punctuation-normalized matches.

Canonical vocabulary identity:

- `lemma` is the normalized dictionary form used for display and lookup.
- `pos` is the normalized part of speech, not raw model output.
- Planned canonical `wordKey` rule: `{normalized lemma}_{normalized pos}`.
- Current runtime implementation stores `lemmaId` and validates
  `wordKey === {lemmaId}_{pos}`.
- Today, generated `lemmaId` is derived from normalized `lemma`, so the planned
  rule and current runtime rule are intended to be equivalent for generated
  content.
- Do not remove `lemmaId` or change `wordKey` semantics without a migration and
  validator update.
- JSON Schema validates the structural shape and enum values for vocabulary
  items. Runtime validation enforces cross-field identity rules such as
  `wordKey === {lemmaId}_{pos}`.
- Normalization is responsible for producing `lemmaId`, normalized `pos`, and
  `wordKey` before validation.

Current strict CEFR enum:

`A1 | A2 | B1 | B2 | C1 | C2`

Difficulty provenance:

- `level` remains the learner-facing CEFR level.
- `levelSource` is optional provenance for the selected CEFR level.
- `difficultyScore` is optional advisory numeric difficulty; it does not replace
  `level`.
- `difficultySource` is optional provenance for `difficultyScore`.
- `difficulty_heuristic` is the canonical fallback source label. Plain
  `heuristic` is not accepted, to avoid confusion with the Debug UI heuristics
  checkbox.
- Raw Zipf values, frequency evidence, `currentDifficulty` /
  `shadowDifficulty`, and resolver traces remain debug-only or future
  repository-index metadata. They are not canonical lesson JSON fields.
- Runtime validation and JSON Schema support `levelSource` and
  `difficultySource`, but current generation does not emit them automatically
  yet.

Planned stabilization fields:

- `lemmaId`: keep as a stable normalized lemma identifier unless a future
  migration replaces it explicitly.
- `difficultyScore`: optional today; intended to become a stable advisory
  numeric difficulty signal once the scoring model is finalized.

Normalization details from `src/schema/normalization.ts`:

- raw `type` or `pos` values are normalized into `pos`
- raw `adj` becomes `adjective`
- raw `adv` becomes `adverb`
- unknown part-of-speech values fall back to `phrase`
- raw CEFR values are normalized:
- `B1+` becomes `B2`
- `B2+` becomes `C1`
- unknown level values fall back to `B2`
- `lemmaId` is lowercased and normalized from `lemma`
- `wordKey` is always `{lemmaId}_{pos}`
- `exampleText` is normalized into `example`

Translations structure:

```json
{
  "translations": {
    "<lang>": {
      "translation": "string"
    }
  }
}
```

Notes:

- Current validation requires every `translations[lang]` value to be an object with a non-empty `translation` string.
- The TypeScript type still allows `string | { translation: string }`, but normalized/generated data and validation use the object form.
- The legacy raw input field `translation` is converted into `translations.ru.translation` during normalization.

# Named Entity Schema

Current runtime lessons may include a separate `namedEntities` array:

```json
{
  "entityKey": "diana_wynne_jones_person",
  "text": "Diana Wynne Jones",
  "normalizedText": "diana wynne jones",
  "entityType": "PERSON",
  "spacyLabel": "PERSON",
  "entityCategory": "character_candidate",
  "learningType": "named_entity",
  "difficultyMarker": "proper_name",
  "source": "spacy_ner",
  "start": 0,
  "end": 17,
  "tokenIndexes": [0, 1, 2]
}
```

Named entity rules:

- `namedEntities` are separate from `vocabulary`.
- Named entities do not receive CEFR `level`.
- `learningType` is always `named_entity`.
- `difficultyMarker` is `proper_name` or `entity_value`.
- `entityKey` is deterministic: normalized entity text plus normalized entity
  type, for example `london_gpe`.
- `start` and `end` are offsets into final `lesson.text`.
- Runtime validation requires `lesson.text.slice(start, end) === text`.

JSON Schema support:

- Runtime `validateLesson` supports `namedEntities`.
- `schemas/lesson.schema.json` includes optional `namedEntities`.
- Debug-only review, recommendation, and dry-run learning-decision fields remain
  excluded from canonical lesson JSON.

# Language Model

Source language model in the current pipeline:

- book source language is carried as `options.language` in `generationPipeline`
- `buildLesson` stores that value in `meta.bookLanguage`
- book manifests store the same concept as `book.json.language`

Current language responsibilities:

- `book.json.language` is the book source language
- `lesson.meta.bookLanguage` is the lesson source language
- `word`, `ipa`, `definition`, and `example` are intended to stay in the source language
- `translations` contain target-language translations only

Prompt behavior:

- the OpenAI lesson prompt includes `Source language of the text is: {language}`
- the prompt instructs the model that words, IPA, definitions, and examples must remain in the source language
- the prompt instructs the model not to mix source language content with translation languages

# Validation Rules

Validation comes from both `src/schema/validate.ts` and `schemas/lesson.schema.json`.

Lesson-level rules:

- `id` is required and must be non-empty
- `schemaVersion` must equal `1`
- `fragmentId` is required and must start with `sha1`
- `generationHash` is required and must start with `sha1`
- `text` is required
- `vocabulary` must be an array
- `meta.bookLanguage` is required and non-empty
- `meta.stats.levelDistribution` must contain all of `A1`, `A2`, `B1`, `B2`, `C1`, `C2`
- each level-distribution value must be a finite number in runtime validation
- JSON Schema further constrains the generated file shape and disallows additional properties

Vocabulary rules:

- `word` is required and non-empty
- `lemma` is required and non-empty
- `lemmaId` is required and non-empty
- `pos` must be one of `noun`, `verb`, `adjective`, `adverb`, `phrase`, `phrasal_verb`, `prepositional_verb`
- optional `type` must be one of `word`, `phrase`, `phrasal_verb`, `prepositional_verb`
- optional `source` must be one of `dictionary_phrasal`, `spacy_phrase`, `fixed_expression`, `word`
- `level` must be one of `A1`, `A2`, `B1`, `B2`, `C1`, `C2`
- optional `levelSource` must be one of `vocabulary_index`,
  `cefr_frequency`, `phrase_index`, `dictionary_phrase`,
  `head_verb_index`, `phrase_fallback`, `difficulty_heuristic`
- optional `difficultySource` must be one of `zipf`, `cefr_frequency`,
  `vocabulary_index`, `difficulty_heuristic`, `none`
- `wordKey` must equal `{lemmaId}_{pos}`
- `translations` must exist
- every translation key must be non-empty
- every `translations[lang]` must be an object
- every `translations[lang].translation` must be non-empty

Language-signal rules:

- if `example` is present and `word` is non-empty, `example` must contain `word`
- current runtime requires ordinary single-token vocabulary items to have
  `matchedForm` when `example` is present
- phrase-like vocabulary items may satisfy example alignment with either
  single-token `matchedForm` or valid `phraseMatch`
- `matchedForm` must be a single token; this runtime cross-field rule is not
  fully expressible in the current JSON Schema shape-only file
- optional `phraseMatch`, when present, must be on a phrase-like item and its
  offsets must match `phraseMatch.text` inside `example`
- when `exampleStart` or `exampleEnd` is present, both must be present and must
  match the exact `lesson.text` substring for `example`
- if `ipa` is present, it must not contain digits
- if `ipa` is present, it must pass the current basic language-signal check
- if `definition` is present, it must match the book-language script signal
- if `example` is present, it must match the book-language script signal

Important precision note:

- `ipa` is optional in the current schema, not required
- when present, runtime validation effectively requires it to be non-empty because it is trimmed and checked
- `definition` and `example` are optional in the current schema
- `pos` normalization happens during vocabulary normalization, not inside `validate.ts`

Named entity rules:

- `namedEntities`, when present, must be an array
- each item must have `entityKey`, `text`, `normalizedText`, `entityType`,
  `spacyLabel`, `entityCategory`, `learningType`, `difficultyMarker`, `source`,
  `start`, `end`, and non-empty `tokenIndexes`
- `learningType` must equal `named_entity`
- `source` must equal `spacy_ner`
- `entityCategory` and `difficultyMarker` must match deterministic mappings
  from `entityType`
- `entityKey` must equal the deterministic key generated from
  `normalizedText` and `entityType`
- `level` must not be set
- `start` and `end` must point into `lesson.text`, and the substring must
  exactly equal `text`

# Determinism Rules

Current determinism behavior:

- `schemaVersion` is fixed at `1`
- `fragmentId` is deterministic from normalized fragment text
- `generationHash` is deterministic from generation config, including language, fragment size, target languages, schema version, and pipeline version
- fragment processing order is stable: first by chapter number, then by fragment index in chapter
- vocabulary items inside a lesson are sorted by `wordKey`
- target languages are sorted before being included in generation config
- repository output paths are deterministic from series, book, section, and lesson ids

Text normalization used before hashing:

- `\r\n` and `\r` are normalized to `\n`
- trailing whitespace is removed from each line
- outer whitespace is trimmed

ID determinism:

- current generated ids use `sct`
- legacy `ch` ids can still appear when legacy chapter input fields are used with `buildLesson`

# Word / Vocabulary Index Entry

The repository vocabulary index is derived from canonical lesson vocabulary. It
is not a separate source of truth.

Current index entries group occurrences under lemma-level records and preserve
word/POS-specific records beneath each lemma. The stable concepts are:

- `schemaVersion: 1`
- source `language`
- lemma key / `lemmaId`
- `lemma`
- `words` keyed by `wordKey`
- normalized `pos`
- CEFR `level`
- occurrence records with `bookId`, section/chapter position, and
  `fragmentId`

Identifier rules:

- Use the same normalized lemma and normalized POS identity as `VocabularyItem`.
- `wordKey` must remain deterministic across regenerated books.
- Occurrence ordering must be deterministic by book/section/fragment order.
- Indexes are rebuildable derived content; user progress must not be stored in
  them.

# Content Data vs User Progress

Generated content is immutable project data. User progress is mutable user data.

Content data includes:

- catalog, book, section, and lesson manifests
- lesson text and `lesson.vocabulary`
- generated named entities once/if they are part of the canonical lesson schema
- derived vocabulary indexes

User progress includes:

- learned/known state
- review scheduling state
- per-user analytics
- notes, highlights, or learner-specific overrides

Progress should reference stable content identifiers such as `wordKey`,
`fragmentId`, `lesson.id`, and book identifiers. Progress must not be written
back into generated lesson JSON or repository vocabulary indexes.

# Debug Metadata Is Not Canonical Content

Full-book regression and debug UI diagnostics can expose rich metadata that is
useful for QA, including:

- Named Entity review status and reasons
- Named Entity to vocabulary overlap diagnostics
- dry-run `learningDecision`
- phrasal verb audit and review cleanup diagnostics
- performance, benchmark, and quality-gate summaries

These diagnostics are reporting-only unless a future schema migration explicitly
adds them to canonical content. In particular, dry-run `learningDecision` must
not filter vocabulary and must not be stored in canonical lesson JSON.

# Example

Trimmed example matching the current canonical generated shape:

```json
{
  "id": "charmed_life_sct1_f1",
  "schemaVersion": 1,
  "fragmentId": "sha1:abc",
  "generationHash": "sha1:confighash",
  "title": "Charmed Life - Chapter 1 (Fragment 1)",
  "text": "Hello there.",
  "meta": {
    "bookId": "charmed_life",
    "bookTitle": "Charmed Life",
    "bookLanguage": "en",
    "author": "Diana Wynne Jones",
    "seriesId": "chrestomanci",
    "seriesTitle": "Chrestomanci",
    "sectionIndex": 1,
    "sectionTitle": "Chapter 1",
    "fragmentIndex": 1,
    "fragmentsInSection": 1,
    "sourceFile": "book.xhtml",
    "stats": {
      "wordCount": 2,
      "uniqueWordCount": 2,
      "paragraphCount": 1,
      "vocabularyCount": 1,
      "levelDistribution": {
        "A1": 1,
        "A2": 0,
        "B1": 0,
        "B2": 0,
        "C1": 0,
        "C2": 0
      }
    }
  },
  "vocabulary": [
    {
      "word": "hello",
      "lemma": "hello",
      "lemmaId": "hello",
      "pos": "phrase",
      "wordKey": "hello_phrase",
      "level": "A1",
      "translations": {
        "uk": {
          "translation": "privit"
        }
      }
    }
  ],
  "namedEntities": [
    {
      "entityKey": "diana_wynne_jones_person",
      "text": "Diana Wynne Jones",
      "normalizedText": "diana wynne jones",
      "entityType": "PERSON",
      "spacyLabel": "PERSON",
      "entityCategory": "character_candidate",
      "learningType": "named_entity",
      "difficultyMarker": "proper_name",
      "source": "spacy_ner",
      "start": 0,
      "end": 17,
      "tokenIndexes": [0, 1, 2]
    }
  ]
}
```

Example provenance:

- structure from `buildLesson.ts`, `types.ts`, `validate.ts`, and `lesson.schema.json`
- field values adapted from repository validation fixtures and lesson schema tests
- trimmed to the canonical generated `section*` form used by the current pipeline
- `namedEntities` shown from current runtime TypeScript validation and
  `schemas/lesson.schema.json`
