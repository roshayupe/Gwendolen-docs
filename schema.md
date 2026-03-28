# Overview

This document describes the current lesson data structures used by EPUB AI Importer.

Sources used:

- `src/lesson/buildLesson.ts`
- `src/schema/validate.ts`
- `src/pipeline/generationPipeline.ts`
- `src/schema/types.ts`
- `src/schema/normalization.ts`
- `schemas/lesson.schema.json`
- checked-in lesson JSON fixtures under `content/` and repository validation tests

This document is intentionally strict. It reflects the current implementation, including compatibility rules, and does not introduce new fields or nested objects that are not present in code.

# Lesson Schema

Current generated lesson objects have this exact top-level structure:

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
  "vocabulary": []
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

Exact generation behavior:

- `id` is generated as `{bookId}_sct{sectionIndex}_f{fragmentIndex}` by current `buildLesson`.
- `fragmentId` is `sha1:` plus the SHA-1 hash of normalized fragment text.
- `generationHash` is produced from generation config in `generationPipeline`.
- `title` is `{bookTitle} - {sectionTitle || "Section N"} (Fragment {fragmentIndex})`.
- `text` is normalized fragment text with normalized line endings, trailing whitespace removed per line, and outer whitespace trimmed.

`meta` is flat. There is no current generated structure like `meta.book`, `meta.chapter`, or `meta.fragment`.

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
  "pos": "noun | verb | adjective | adverb | phrase",
  "wordKey": "string",
  "level": "A1 | A2 | B1 | B2 | C1 | C2",
  "difficultyScore": 0.0,
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

Field list:

- `word: string` required
- `lemma: string` required
- `lemmaId: string` required
- `pos: "noun" | "verb" | "adjective" | "adverb" | "phrase"` required
- `wordKey: string` required
- `level: "A1" | "A2" | "B1" | "B2" | "C1" | "C2"` required
- `difficultyScore?: number`
- `ipa?: string`
- `translations: Record<string, { translation: string }>` required at runtime
- `definition?: string`
- `example?: string`

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
- `pos` must be one of `noun`, `verb`, `adjective`, `adverb`, `phrase`
- `level` must be one of `A1`, `A2`, `B1`, `B2`, `C1`, `C2`
- `wordKey` must equal `{lemmaId}_{pos}`
- `translations` must exist
- every translation key must be non-empty
- every `translations[lang]` must be an object
- every `translations[lang].translation` must be non-empty

Language-signal rules:

- if `example` is present and `word` is non-empty, `example` must contain `word`
- if `ipa` is present, it must not contain digits
- if `ipa` is present, it must pass the current basic language-signal check
- if `definition` is present, it must match the book-language script signal
- if `example` is present, it must match the book-language script signal

Important precision note:

- `ipa` is optional in the current schema, not required
- when present, runtime validation effectively requires it to be non-empty because it is trimmed and checked
- `definition` and `example` are optional in the current schema
- `pos` normalization happens during vocabulary normalization, not inside `validate.ts`

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
  ]
}
```

Example provenance:

- structure from `buildLesson.ts`, `types.ts`, and `lesson.schema.json`
- field values adapted from repository validation fixtures and lesson schema tests
- trimmed to the canonical generated `section*` form used by the current pipeline
