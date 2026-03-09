# Project Roadmap

This document describes the long-term development roadmap of the AI-powered language learning system.

The project evolves through three major stages:

1. Solid MVP
2. Shareable Product
3. SaaS Platform

Each stage expands the system capabilities while preserving architectural stability.

---

# Stage 1 — Solid MVP

Goal:

Create a stable foundation for content generation and reading.

The system must be predictable, robust, and resistant to AI generation errors.

Core capabilities:

• EPUB parsing  
• Fragment-based lesson generation  
• Vocabulary cards with translation and examples  
• Reader interface for studying fragments  
• Local progress tracking  
• Basic lesson navigation

Infrastructure introduced:

• lesson JSON structure  
• reader interface  
• basic generation pipeline  
• local progress storage

Result:

A working prototype capable of generating and reading vocabulary lessons from books.

---

# Stage 2 — Shareable Product

Goal:

Transform the prototype into a stable and distributable product.

Content generation becomes automated and the reader gains learning analytics.

Major improvements:

### Schema Stabilization
Define canonical data structures used by generator, assembler, and reader.

### Repository Structure
Introduce deterministic repository layout for books, chapters, and lessons.

### Assembler Engine
Convert generated fragments into repository-ready book structures.

### Cloudflare Worker Safety
Ensure the generation system runs safely and cost-efficiently.

### Generation Pipeline
Create a fully automated workflow:

EPUB → AI generation → lesson fragments → assembler → repository commit.

### Reader Intelligence
Turn the reader into a learning tool by adding:

• vocabulary progress tracking  
• fragment difficulty calculation  
• chapter and book progress analytics  
• progress export/import  
• vocabulary indexing

Result:

A stable content engine capable of generating a growing library of vocabulary-enhanced books.

---

# Stage 3 — SaaS Platform

Goal:

Turn the system into a scalable cloud learning platform.

Key features:

### User Accounts
Cloud-based progress storage and multi-device synchronization.

### Vocabulary Knowledge Graph
Global vocabulary graph linking words, fragments, and books.

### Personalized Reading
AI-driven fragment recommendations based on learner knowledge.

### Global Library
Large-scale repository of automatically generated books.

### Subscription Model
Access to the full learning platform as a service.

Result:

A scalable AI-powered language learning platform capable of supporting thousands of books and users.

---

# Long-Term Vision

The system becomes an adaptive language learning engine where:

• books generate structured vocabulary lessons automatically  
• readers study real literature instead of artificial exercises  
• the platform recommends optimal reading material for each learner  

This approach combines:

AI content processing  
vocabulary learning science  
large-scale digital libraries