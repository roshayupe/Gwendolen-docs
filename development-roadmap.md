# EPUB AI Importer — Development Roadmap

This document defines the development roadmap for the EPUB AI Importer project.

The roadmap is designed to guarantee:

• deterministic generation  
• repository-friendly content  
• schema stability  
• scalable vocabulary architecture  

The system must support **tens of thousands of books** without schema redesign.

---

# Milestone Overview

| Milestone | Name | Status |
|-----------|------|--------|
| 0 | Test Infrastructure | ✅ done |
| 1 | Golden Book Fragment Engine | ✅ done |
| 2 | Schema Stabilization | ⏳ current |
| 3 | Vocabulary Normalization | planned |
| 4 | Vocabulary Deduplication | planned |
| 5 | Global Vocabulary Index | planned |
| 6 | Vocabulary Cache Engine | planned |
| 7 | Deterministic Lesson Builder | planned |
| 8 | Large-Scale Book Processing | planned |

---

# Milestone 2 — Schema Stabilization

Goal:

Freeze the canonical lesson schema.

Files involved:


src/schema/types.ts
src/schema/validate.ts
tests/schema/*


---

## Task 2.1 — Implement TypeScript schema

File:


src/schema/types.ts


Prompt:


Implement canonical TypeScript interfaces for the lesson schema.

Requirements:

The schema must follow the structure defined in schema.md.

Interfaces required:

CefrLevel
Pos
VocabularyItem
LessonStats
LessonMeta
Lesson

Rules:

• schemaVersion must equal 1
• vocabulary must be VocabularyItem[]
• levelDistribution must be Record<CefrLevel, number>

Do not redesign the schema.
Follow schema.md exactly.


---

## Task 2.2 — Implement schema validator

File:


src/schema/validate.ts


Prompt:


Create a deterministic validator for Lesson JSON objects.

The validator must:

• validate vocabulary items
• validate CEFR levels
• validate part of speech
• verify wordKey format
• verify fragmentId format

Throw explicit errors when validation fails.

Do not use external libraries.


---

## Task 2.3 — Add schema tests

Files:


tests/schema/lessonSchema.test.ts
tests/schema/vocabularySchema.test.ts


Prompt:


Create Vitest tests validating the canonical lesson schema.

Tests must verify:

• schemaVersion exists
• fragmentId starts with sha1
• vocabulary items follow canonical schema
• CEFR levels are valid
• wordKey equals lemmaId + "_" + pos

Tests must be deterministic.


---

# Milestone 3 — Vocabulary Normalization

Goal:

Normalize AI output to canonical vocabulary schema.

File:


src/schema/normalization.ts


Prompt:


Implement normalization functions converting raw AI vocabulary output
to canonical VocabularyItem objects.

Functions required:

normalizeLemmaId
normalizeLevel
normalizePos
buildWordKey
normalizeVocabularyItem

Rules:

B1+ → B2
B2+ → C1

adj → adjective
adv → adverb

lemmaId must be lowercase with underscores.

wordKey format:

lemmaId + "_" + pos


---

# Milestone 4 — Vocabulary Deduplication

Goal:

Remove duplicate vocabulary items produced by AI.

File:


src/vocabulary/deduplicateVocabulary.ts


Prompt:


Create a vocabulary deduplication engine.

Input:

array of raw vocabulary objects.

Output:

canonical VocabularyItem[].

Rules:

Duplicate detection must use:

wordKey = lemmaId + "_" + pos

If duplicates exist:

• merge translations
• keep longest definition
• keep longest example
• keep highest difficultyScore

Output must be sorted by wordKey.


---

# Milestone 5 — Global Vocabulary Index

Goal:

Build a global vocabulary index across all lessons.

File:


src/vocabulary/buildVocabularyIndex.ts


Prompt:


Implement a vocabulary index builder.

Input:

Lesson[].

Output:

VocabularyIndex.

VocabularyIndexEntry must contain:

lemma
pos
level
translations
definition
occurrences
occurrenceCount

Rules:

• occurrences must be sorted
• vocabulary keys must be wordKey
• the index must be deterministic


---

# Milestone 6 — Vocabulary Cache Engine

Goal:

Avoid regenerating vocabulary that already exists.

File:


src/vocabulary/vocabularyCache.ts


Prompt:


Implement a vocabulary cache engine.

The cache must:

• store vocabulary by wordKey
• detect if a word already exists
• reuse existing definition and translations
• avoid regenerating the same vocabulary item via AI

The cache must be deterministic and file-based.


---

# Milestone 7 — Deterministic Lesson Builder

Goal:

Create canonical lesson JSON files.

File:


src/lesson/buildLesson.ts


Prompt:


Implement a deterministic lesson builder.

Input:

fragment text
chapter metadata
normalized vocabulary

Output:

Lesson JSON.

Rules:

• generate fragmentId using sha1(fragmentText)
• build id as bookId_ch{chapter}_f{fragment}
• compute lesson statistics
• sort vocabulary by wordKey


---

# Milestone 8 — Large-Scale Book Processing

Goal:

Enable processing of thousands of books.

Prompt:


Design a scalable pipeline for processing thousands of EPUB files.

Requirements:

• incremental processing
• fragment hashing
• vocabulary cache
• deterministic outputs
• ability to skip already processed fragments


---

# AI Development Rules

When using AI tools (ChatGPT / Codex):

Always include:


Follow schema.md exactly.
Do not redesign the architecture.
Preserve deterministic output.


---

# Determinism Requirements

All generation must be deterministic.

Forbidden in canonical content:


generatedAt
timestamp
model
requestId


---

# Snapshot Tests

Golden tests must always pass.

To update snapshots intentionally:


UPDATE_GOLDEN=1 npm test


---

# Long-Term Goal

The architecture must support:


10 000+ books
global vocabulary index
deterministic lesson generation


---