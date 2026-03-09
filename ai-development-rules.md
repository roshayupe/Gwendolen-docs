# AI Development Rules

This document defines rules that AI assistants (Codex, GPT, etc.) must follow when modifying the repository.

The goal is to preserve architectural integrity and prevent accidental system corruption.

AI tools must treat this document as **binding development constraints**.

---

# Core Principles

1. Content is immutable.
2. User progress is stored separately.
3. Repository structure must remain deterministic.
4. Vocabulary identity must remain stable.
5. Schema changes require explicit version updates.

---

# Architectural Boundaries

The system consists of several independent subsystems.

AI must not mix responsibilities between them.

Subsystems:

• Content Generation  
• Assembler  
• Repository Content  
• Reader Application  
• User Progress Layer  

Each subsystem must remain isolated.

---

# Content Rules

AI must **never modify existing lesson content** unless explicitly instructed.

Forbidden operations:

- modifying vocabulary entries inside existing lessons
- changing fragment text
- adding progress fields to lesson JSON
- rewriting existing generated books

Allowed operations:

- generating new books
- generating new fragments
- updating indexes
- assembling new repository structures

---

# Progress Rules

User progress must never be written into lesson files.

Forbidden fields in lesson JSON:
mastered
known
progress
userState


Progress must always be stored separately.

Example progress storage:

```json
{
  "survive_verb": true,
  "determine_verb": true
}

Progress is indexed by wordKey.

Vocabulary Identity Rules

Vocabulary identity must remain stable.

Definition:

wordKey = lemma + "_" + pos

Examples:

survive_verb
book_noun
turn_down_phrase

AI must never change the definition of wordKey.

Schema Rules

All lesson JSON files must follow the schema defined in:

docs/schema.md

AI must not introduce new fields unless:

schemaVersion is increased

schema.md is updated

Repository Structure Rules

Generated content must follow the canonical structure:

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

AI must not introduce alternative layouts.

Fragment Identity Rules

Fragment identifiers must remain deterministic.

Format:

bookId_ch<chapterIndex>_f<fragmentIndex>_<hash>

The hash must be derived from fragment text.

This guarantees fragment stability across regeneration.

Generation Pipeline Rules

Generation pipeline responsibilities:

parse EPUB

split chapters into fragments

generate vocabulary

create lesson JSON

assemble repository structure

commit generated content

AI must not combine unrelated responsibilities inside one module.

Example:

Assembler must not parse EPUB files.

Safety Rules

AI-generated code must:

avoid destructive repository changes

avoid deleting generated content

preserve existing manifests

validate repository structure before writing files

Allowed Refactoring

AI may perform refactoring that does not change behavior:

code formatting

internal module restructuring

performance improvements

adding validation

AI must not change external data contracts.

Schema Evolution

Schema changes must follow this process:

Update schema.md

Increment schemaVersion

Add migration logic if needed

Update generator and reader

Skipping this process may corrupt the content library.

Large Library Considerations

The system is designed for very large libraries.

Expected scale:

• Books: 75,000+
• Fragments: millions
• Vocabulary entries: tens of millions

AI implementations must prioritize:

deterministic generation

stable identifiers

efficient indexing

When AI Is Uncertain

If an AI assistant is uncertain about an architectural change, it must:

Check system-architecture.md

Check schema.md

Follow implementation-order.md

Avoid modifying content files

Summary

AI assistants must respect:

repository structure

schema stability

vocabulary identity

content immutability

subsystem boundaries