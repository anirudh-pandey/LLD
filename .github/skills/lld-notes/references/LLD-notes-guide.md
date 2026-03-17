# LLD Notes Creation Guide

Use this file as the reference for creating any LLD revision notes (OOP, SOLID, Design Patterns, etc.).

## File Location
Place notes inside the relevant subfolder under `LLD/`. For example:
- OOP topics → `LLD/OOPS/<topic>.md`
- SOLID topics → `LLD/SOLID/<topic>.md`
- Design Patterns → `LLD/design-patterns/<topic>.md`

## Language
- Use Java for all code examples.

## Tone & Style
- Write like you are explaining to a friend, not writing a textbook.
- Use short paragraphs instead of long bullet lists. One idea per paragraph.
- Avoid dry, formal language. Keep it conversational but precise.
- Do not overload the reader. Each section should feel light to read.

## Structure (follow this order)

### 1. In One Line
A single-sentence summary of the concept in plain words.

### 2. Core Idea
2-3 short paragraphs explaining the concept simply. No bullet lists here.

### 3. Why Do We Need It?
Explain motivation in 2-3 short paragraphs with a small real example woven in.
Do not use a long list of reasons. Instead connect the ideas naturally.

### 4. What Goes Wrong Without It?
2-3 short paragraphs showing concrete failure scenarios.
Mention specific examples like "a bank account with negative balance" or "an order skipping status transitions."

### 5. Simple Example
Show a **bad design** code snippet and explain what is wrong in 1-2 lines.
Then show a **better design** code snippet and explain what improved.
Keep code short. 10-20 lines max per snippet.

### 6. Another Easy Example (optional)
A second example, ideally a real-world analogy (coffee machine, ATM, car steering) or a different code scenario.

### 7. Real Meaning
Clarify what the concept is NOT (common misconceptions).
Then state what it truly means in 2-3 short lines.

### 8. Good vs Bad Example (if applicable)
Show a bad usage of the concept vs a proper one to highlight the difference.

### 9. Comparison With Related Concept (if applicable)
If there is a commonly confused pair (e.g., abstraction vs encapsulation), add a short comparison.
Use a simple memory trick or one-liner to help recall the difference.

### 10. Strong Mental Model
Give one question the reader can ask themselves to check if they are applying the concept correctly.

### 11. Interview-Ready Definition
One polished paragraph (3-4 lines) that could be spoken directly in an interview.

### 12. Interview-Ready Why
One polished paragraph (2-3 lines) answering "why do we need this?"

### 13. Fast Recall
3 short "Think: ..." lines that act as quick mental triggers.

### 14. Questions To Ask In Design
4 short questions (one per line, no bullets) the reader should ask themselves when designing a system.

## Formatting Rules
- Use `##` for section headings.
- Use ```java for code blocks.
- Separate paragraphs with blank lines.
- No nested bullet lists.
- Bullet lists only where a short list of 3-5 items is genuinely the clearest format (e.g., Real Meaning misconceptions).
- Keep the total note under ~120 lines. This is a revision sheet, not a textbook chapter.
