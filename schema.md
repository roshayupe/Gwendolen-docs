# EPUB AI Importer --- Canonical Data Schema

Version: 1

This document defines the canonical JSON schema used by the EPUB AI
Importer pipeline.

The schema is designed to support:

• deterministic generation\
• repository-friendly content storage\
• large-scale book libraries\
• stable vocabulary indexing\
• AI processing pipelines

Content data must be immutable.\
User progress must always be stored separately.

------------------------------------------------------------------------

# Pipeline Context

EPUB\
↓\
EPUB parser\
↓\
chapter extraction\
↓\
fragment generator\
↓\
lesson generator\
↓\
vocabulary extraction\
↓\
content assembler\
↓\
reader UI

Each fragment becomes **one lesson JSON file**.

------------------------------------------------------------------------

# Determinism Requirements

Generated lesson JSON files must be deterministic.

The following fields **must NOT appear in canonical content JSON**:

-   timestamps
-   request IDs
-   AI model identifiers
-   generation metadata

Forbidden examples:

generatedAt\
timestamp\
requestId\
model

------------------------------------------------------------------------

# Canonical Lesson Structure

Example:

``` json
{
  "id": "the_magicians_of_caprona_ch1_f1",
  "schemaVersion": 1,
  "fragmentId": "sha1:3d9d5c7d3d5f0e3f6d7a1b8c9e0f123456789abc",
  "title": "The Magicians of Caprona — Chapter 1 (Fragment 1)",
  "text": "Fragment source text here...",
  "meta": {
    "bookId": "the_magicians_of_caprona",
    "bookTitle": "The Magicians of Caprona",
    "author": "Jones, Diana Wynne",
    "seriesId": "chrestomanci_series",
    "seriesTitle": "Chrestomanci Series",
    "chapterIndex": 1,
    "chapterTitle": "Chapter 1",
    "fragmentIndex": 1,
    "fragmentsInChapter": 4,
    "sourceFile": "OEBPS/005_chapter1.htm",
    "stats": {
      "wordCount": 753,
      "uniqueWordCount": 341,
      "paragraphCount": 12,
      "vocabularyCount": 28,
      "levelDistribution": {
        "A1": 0,
        "A2": 0,
        "B1": 0,
        "B2": 15,
        "C1": 11,
        "C2": 2
      }
    }
  },
  "vocabulary": []
}
```

------------------------------------------------------------------------

# Lesson Field Definitions

## id

Deterministic lesson identifier.

Format:

    {bookId}_ch{chapterIndex}_f{fragmentIndex}

Example:

    the_magicians_of_caprona_ch1_f1

------------------------------------------------------------------------

## schemaVersion

Schema version number.

Current version:

    1

All schema changes must increment this number.

------------------------------------------------------------------------

## fragmentId

Deterministic identifier derived from fragment text.

Format:

    sha1(normalizedFragmentText)

Purpose:

-   duplicate detection
-   incremental regeneration
-   stable fragment references

------------------------------------------------------------------------

## text

Clean fragment text extracted from EPUB.

Rules:

-   paragraphs preserved
-   normalized whitespace
-   normalized line endings

------------------------------------------------------------------------

# Metadata

Metadata describing the fragment source.

Fields:

-   bookId
-   bookTitle
-   author
-   seriesId
-   seriesTitle
-   chapterIndex
-   chapterTitle
-   fragmentIndex
-   fragmentsInChapter
-   sourceFile

------------------------------------------------------------------------

# Lesson Statistics

Stored in:

    meta.stats

Example:

``` json
{
  "wordCount": 753,
  "uniqueWordCount": 341,
  "paragraphCount": 12,
  "vocabularyCount": 28,
  "levelDistribution": {
    "A1": 0,
    "A2": 0,
    "B1": 0,
    "B2": 15,
    "C1": 11,
    "C2": 2
  }
}
```

------------------------------------------------------------------------

# Vocabulary Model

Example vocabulary entry:

``` json
{
  "word": "captious",
  "lemma": "captious",
  "lemmaId": "captious",
  "pos": "adjective",
  "wordKey": "captious_adjective",
  "level": "C1",
  "difficultyScore": 0.86,
  "ipa": "ˈkæpʃəs",
  "translations": {
    "ru": "придирчивый"
  },
  "definition": "Tending to find fault or raise trivial objections.",
  "example": "Though it remained bad-tempered, captious, and unfriendly."
}
```

------------------------------------------------------------------------

# Vocabulary Fields

## word

Surface form appearing in the text.

------------------------------------------------------------------------

## lemma

Dictionary base form.

Examples:

running → run\
troubles → trouble

------------------------------------------------------------------------

## lemmaId

Normalized identifier derived from lemma.

Rules:

-   lowercase
-   spaces → `_`
-   punctuation removed

Example:

love charm → love_charm

------------------------------------------------------------------------

## pos

Allowed values:

noun\
verb\
adjective\
adverb\
phrase

------------------------------------------------------------------------

## wordKey

Stable vocabulary identifier.

Format:

    lemmaId + "_" + pos

Examples:

run_verb\
run_noun\
captious_adjective

Purpose:

-   vocabulary indexing across books
-   user progress tracking

------------------------------------------------------------------------

## level

CEFR difficulty level.

Allowed values:

A1\
A2\
B1\
B2\
C1\
C2

Normalization rules:

B1+ → B2\
B2+ → C1

------------------------------------------------------------------------

## difficultyScore

Numeric difficulty estimate.

Range:

0.0 -- 1.0

Used for:

-   ranking
-   adaptive learning

------------------------------------------------------------------------

## ipa

International Phonetic Alphabet pronunciation.

Optional.

------------------------------------------------------------------------

## translations

Dictionary of translations.

Example:

``` json
{
  "ru": "придирчивый",
  "uk": "прискіпливий"
}
```

------------------------------------------------------------------------

## definition

Short dictionary definition.

Optional.

------------------------------------------------------------------------

## example

Example sentence demonstrating usage.

Optional.

------------------------------------------------------------------------

# Fragment Identity

Algorithm:

    fragmentId = sha1(normalize(fragmentText))

Normalization:

-   trim whitespace
-   normalize line endings
-   collapse trailing spaces

------------------------------------------------------------------------

# Word Identity

Rule:

    wordKey = lemmaId + "_" + pos

Example:

    captious_adjective

------------------------------------------------------------------------

# Content vs User Data

Content JSON must be immutable.

Recommended structure:

    content/
      book/
        chapter/
          lesson.json

    user/
      progress.json

------------------------------------------------------------------------

# Example User Progress

``` json
{
  "schemaVersion": 1,
  "lessonId": "the_magicians_of_caprona_ch1_f1",
  "masteredWordKeys": [
    "captious_adjective",
    "feast_noun"
  ]
}
```

------------------------------------------------------------------------

# Testing Requirements

Recommended tests:

tests/schema/lessonSchema.test.ts\
tests/schema/vocabularySchema.test.ts\
tests/schema/normalization.test.ts

Tests must verify:

-   schemaVersion exists
-   fragmentId exists
-   vocabulary uses canonical fields
-   CEFR levels are valid
-   wordKey is deterministic

------------------------------------------------------------------------

# Schema Evolution

Schema changes must follow rules:

1.  Increment `schemaVersion`
2.  Update `schema.md`
3.  Add schema tests
4.  Update golden snapshots if required

Command:

UPDATE_GOLDEN=1 npm test
