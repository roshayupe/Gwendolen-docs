# Implementation Order — Stage 2 (Shareable Product)

This document defines the **mandatory implementation order** for Stage 2 of the project.

The order must be respected to avoid breaking dependencies between modules.

Each section corresponds to a milestone and lists the issues in the order they should be implemented.

---

# Stage 2 Overview

Stage 2 transforms the project into a **shareable product** by stabilizing the content engine, automating generation, and introducing learning analytics.

Pipeline goal:

EPUB → AI generation → lesson fragments → assembler → repository → reader

---

# Milestone 1 — Schema Stabilization

Goal: Define the canonical data structures used across the system.

Order of implementation:

1. Freeze canonical Lesson JSON structure

2. Introduce `lemma` and `lemmaId` fields

3. Replace `type` with normalized `pos` field

4. Generate stable `wordKey = lemma + "_" + pos`

5. Normalize AI difficulty levels to strict enum

6. Add `difficultyScore` to Word schema

7. Deduplicate vocabulary entries by `wordKey`

8. Add `schemaVersion` to lesson meta

Result:

All generated lessons share a stable schema and vocabulary identity.

---

# Milestone 2 — Repository Structure

Goal: Define the canonical structure of the content repository.

Order of implementation:

1. Define canonical repository folder layout

2. Freeze `book.json` manifest schema

3. Freeze `chapter.json` manifest schema

4. Introduce deterministic `fragmentId` with content hash

5. Add `indexes/` directory for future indexing

6. Implement repository validation script

Result:

Repository structure becomes deterministic and verifiable.

---

# Milestone 3 — Assembler Engine

Goal: Convert generated fragments into repository-ready book structures.

Order of implementation:

1. Create assembler module

2. Validate fragment sequence

3. Generate lesson files

4. Generate chapter manifests

5. Generate book manifest

6. Register book in catalog

7. Add assembler dry-run mode

Result:

Generated fragments can be assembled into a full book structure automatically.

---

# Milestone 4 — Cloudflare Worker Safety

Goal: Ensure the generation worker is safe, reliable, and cost-controlled.

Order of implementation:

1. Introduce generation job abstraction

2. Add generation safety limits

3. Add retry logic for AI calls

4. Prevent concurrent generation for the same book

5. Add structured logging

6. Add timeout protection

Result:

The worker becomes stable and production-safe.

---

# Milestone 5 — Generation Pipeline

Goal: Connect all components into an automated generation pipeline.

Order of implementation:

1. Create pipeline orchestrator

2. Integrate EPUB parser

3. Implement fragment splitting stage

4. Integrate AI vocabulary generation

5. Generate lesson JSON files

6. Integrate assembler stage

7. Implement Git commit stage

8. Add pipeline dry-run mode

Result:

A complete automated workflow:

EPUB → AI → repository commit

---

# Milestone 6 — Reader Intelligence

Goal: Turn the reader into a vocabulary-aware learning system.

Order of implementation:

1. Implement progress storage layer

2. Hide mastered words from lessons

3. Compute fragment difficulty

4. Compute chapter vocabulary statistics

5. Compute book progress

6. Add progress export and import

7. Generate vocabulary lexicon index

8. Add fragment recommendation prototype

Result:

Reader becomes a learning interface rather than just a book viewer.

---

# Stage 2 Completion Criteria

Stage 2 is complete when the system supports:

- Automated EPUB generation pipeline
- Stable repository structure
- Vocabulary-aware reader
- Progress tracking
- Basic learning analytics

At this point the project becomes a **shareable product**.

---

# Stage 3 Preview (SaaS)

Stage 3 will introduce:

- user accounts
- cloud progress storage
- global vocabulary graph
- AI-driven content recommendation
- subscription model