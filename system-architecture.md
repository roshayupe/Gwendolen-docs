# System Architecture

This document describes the architecture of the AI-powered language learning system.

                ┌───────────────┐
                │     EPUB      │
                │   Book File   │
                └───────┬───────┘
                        │
                        ▼
                ┌───────────────┐
                │ EPUB Parser   │
                └───────┬───────┘
                        │
                        ▼
                ┌───────────────┐
                │ Fragment      │
                │ Splitter      │
                └───────┬───────┘
                        │
                        ▼
                ┌───────────────┐
                │ AI Vocabulary │
                │ Generator     │
                └───────┬───────┘
                        │
                        ▼
                ┌───────────────┐
                │ Lesson JSON   │
                │ Fragments     │
                └───────┬───────┘
                        │
                        ▼
                ┌───────────────┐
                │ Assembler     │
                │ (Content      │
                │ Compiler)     │
                └───────┬───────┘
                        │
                        ▼
                ┌───────────────┐
                │ Repository    │
                │ data/         │
                └───────┬───────┘
                        │
                        ▼
                ┌───────────────┐
                │ Reader App    │
                │ Vocabulary UI │
                └───────┬───────┘
                        │
                        ▼
                ┌───────────────┐
                │ User Progress │
                │ (wordKey)     │
                └───────────────┘

The system converts books into vocabulary learning content and provides a reader interface for studying them.

The architecture separates **content generation**, **content storage**, and **user learning state**.

---

# Core System Principle

Content must be immutable.

User progress must be stored separately.

This rule guarantees that the content library can scale independently of user data.

---

# High-Level Architecture

The system consists of four major subsystems:

1. Content Generation
2. Content Repository
3. Reader Application
4. User Progress Layer

Pipeline overview:

EPUB → AI Generation → Lesson Fragments → Assembler → Repository → Reader

---

# Content Generation

Content generation converts books into vocabulary lessons.

Steps:

1. EPUB parsing
2. Chapter extraction
3. Fragment splitting
4. AI vocabulary extraction
5. Lesson creation
6. Assembly into repository structure

The generation system runs inside **Cloudflare Workers**.

Generation flow:

EPUB
↓
Chapter text
↓
Fragments (~1000–1200 words)
↓
AI vocabulary generation
↓
Lesson JSON
↓
Assembler
↓
Repository structure

---

# Repository Structure

All generated content is stored under the `data/` directory.

Structure:

data/
  catalog.json
  collections/
    <collectionId>/
      <bookId>/
        book.json
        chapters/
          ch1/
            chapter.json
            lessons/
              <fragmentId>.json
          ch2/
            chapter.json
            lessons/

  indexes/
    lexicon/
    fragments/

---

# Manifest Hierarchy

Content is connected through manifests.

Hierarchy:

catalog.json
↓
book.json
↓
chapter.json
↓
lesson.json

Each level references the next.

---

# Lesson Structure

Lessons are the atomic learning unit.

Canonical structure:


lesson.json


```json
{
  "meta": {
    "schemaVersion": 1
  },
  "sourceText": "",
  "stats": {},
  "words": []
}

Each lesson corresponds to a text fragment.

Word Schema

Vocabulary items follow a canonical structure.

{
  "word": "",
  "lemma": "",
  "lemmaId": "",
  "pos": "",
  "wordKey": "",

  "level": "",
  "difficultyScore": 0.0,

  "ipa": "",
  "translation": "",
  "definition": "",

  "example": "",
  "exampleText": ""
}

Key fields:

wordKey = lemma + "_" + pos

lemmaId = normalized lemma identifier.

Fragment Identity

Each lesson corresponds to a fragment.

Fragments use deterministic identifiers:

bookId_ch<chapter>f<fragment><hash>

Example:

magicians_of_caprona_ch1_f1_a3d2

Hash is derived from fragment text.

This guarantees stability when regenerating books.

Assembler

The assembler converts generated fragments into repository content.

Responsibilities:

place lesson files in chapter folders

generate chapter manifests

generate book manifest

update catalog

Assembler acts as a content compiler.

Generation Pipeline

The generation pipeline orchestrates the full workflow.

Steps:

parse EPUB

split chapters into fragments

generate vocabulary using AI

create lesson JSON

assemble repository structure

commit book to repository

Pipeline module:

pipeline/runGenerationPipeline.ts

Cloudflare Worker

The generation system runs inside a Worker.

Worker responsibilities:

accept EPUB input

run generation pipeline

enforce safety limits

retry AI calls

prevent concurrent generation

Worker ensures generation jobs are safe and controlled.

Reader Application

The reader loads lesson data from the repository.

Reader responsibilities:

display fragment text

display vocabulary cards

track learning progress

compute learning analytics

Reader does not modify content files.

Progress Storage

User progress is stored separately from content.

Example progress storage:

progress.json
{
  "survive_verb": true,
  "impossible_adjective": true
}

Progress is indexed by wordKey.

Vocabulary Graph

Vocabulary across the entire library forms a graph.

Indexes allow lookup of word occurrences across books.

Example:

indexes/lexicon.json
{
  "survive": [
    "charmed_life_ch1_f1",
    "magicians_of_caprona_ch2_f3"
  ]
}

This supports future recommendation systems.

Difficulty Model

Fragments contain a computed difficulty score.

Example:

stats.difficulty

Difficulty is derived from vocabulary difficulty scores.

This enables adaptive reading.

Learning Engine

Future recommendation logic selects fragments based on unknown vocabulary.

Target example:

5–7 unknown words per fragment.

This produces optimal learning difficulty.

System Scalability

The architecture supports large-scale libraries.

Expected scale:

Books: 75,000+
Fragments: millions
Vocabulary entries: tens of millions

Content is static and CDN-friendly.

User progress remains lightweight.

Stage 2 Scope

Stage 2 delivers:

stable schema

deterministic repository

automated generation pipeline

vocabulary-aware reader

learning progress analytics

This creates a shareable product.

Stage 3 Preview

Stage 3 will introduce:

user accounts

cloud progress sync

personalized recommendations

global vocabulary graph

subscription model