---
name: lld-notes
description: 'Create LLD revision notes for interview prep. Use when asked to create notes, revision sheets, or study material for OOP concepts (encapsulation, abstraction, inheritance, polymorphism), SOLID principles, design patterns, or any other Low Level Design topic. Produces a concise, example-rich markdown note following a consistent structure.'
argument-hint: 'Topic name, e.g. "inheritance" or "strategy pattern"'
---

# LLD Notes Creation

## When to Use
- User asks to create revision notes or a study sheet for an LLD topic.
- User asks to document a concept after a teaching/discussion session.
- User says "create notes for X" where X is an OOP, SOLID, or design pattern topic.

## Procedure

1. **Read the style guide**: Load [LLD-notes-guide.md](./references/LLD-notes-guide.md) and follow it exactly for structure, tone, formatting, and length.

2. **Determine the file location**: Use the guide's file location rules:
   - OOP topics → `OOPS/<topic>.md`
   - SOLID topics → `SOLID/<topic>.md`
   - Design Patterns → `design-patterns/<topic>.md`
   - If the subfolder does not exist, create it.

3. **Write the note**: Follow the section order from the guide. Every note must include:
   - In One Line
   - Core Idea
   - Why Do We Need It?
   - What Goes Wrong Without It?
   - Simple Example (bad design → better design, in Java)
   - Real Meaning
   - Strong Mental Model
   - Interview-Ready Definition
   - Interview-Ready Why
   - Fast Recall
   - Questions To Ask In Design

   Optional sections (include when relevant):
   - Another Easy Example
   - Good vs Bad Example
   - Comparison With Related Concept

4. **Check quality**:
   - Total length under ~120 lines.
   - Short paragraphs, not long bullet lists.
   - Conversational tone, not textbook.
   - Java code snippets kept to 10-20 lines each.
   - No nested bullets. No dry lists of reasons.

5. **Confirm**: Tell the user the file was created and where it lives.

## Reference Examples
Existing notes that follow this style:
- `OOPS/encapsulation.md`
- `OOPS/abstraction.md`
