# Translation System Principles (Always Enforce)

## Do Not Speculate

- Translation systems require high precision; any guessing will cause system problems
- For any rules, formats, or conversion logic that is uncertain, always confirm with the user first
- The source of truth for rules is the Supabase rules table — do not invent rules

## Deterministic Processing First

- Anything that can be handled with deterministic logic (symbol conversion, onomatopoeia lookup tables) should not depend on LLM
- Pre-processing isolation: empty lines/transition lines (skip), onomatopoeia (interjection), pure symbols (symbol)
- Post-processing overrides: half-width punctuation conversion, ellipsis normalization

## Knowledge Base Import Discipline

- glossary: proper nouns, must have chinese + target_text + target_lang
- lore: character settings/world-building, shared across language pairs
- rules: translation rules, differentiated by scope + target_lang
- translation_memory: only store human-corrected translations, never store raw AI translations
- character_interjections: is_onomatopoeia=true for deterministic lookup, false handled by LLM

## Before Modifying Workflows

- Before modifying Dify YAML, verify logic correctness using a local Python test script
- Before modifying Edge Functions, confirm RPC signatures are consistent with SQL migrations
- After modifying Apps Script, list the .gs file names that need to be deployed
