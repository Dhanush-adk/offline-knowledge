# Implementation Notes (from plan validation)

Apply these when implementing; they are reflected in [plan-validation-and-notes.md](plan-validation-and-notes.md) and partially in the main plan.

## Chunking (Section 3 pipeline)

- Use **character-based** chunking: `maxChars: 600`, `overlapChars: 100`.
- Split at sentence/paragraph boundaries when possible (keeps under typical 512-token model limits).
- Token-based chunking can be added later if a tokenizer is introduced.

## Embed step (Section 3)

- Run on a **background queue**; batch in small groups (e.g. 5 chunks) with progress callback for UI (e.g. "Embedding 3 of 12 chunks…").
- Return `[[Float]]` L2-normalized.

## AnswerGenerator (Section 4)

- **Use option (B):** concatenate top chunks with clear section headers and "Source: [title/URL]" labels for better UX.

## Data safety / Export (Section 8)

- Prefer **SQLite copy** for full backup (preserves embeddings).
- "Export library" = write a copy of the DB to a user-chosen location.
- Optional: JSON export (items + chunks without embeddings) for inspection or migration.

## Entitlements (Section 8)

- **Outside Mac App Store:** App can run without sandbox; no network entitlement needed (URLSession works by default). If you enable sandbox anyway, add `com.apple.security.network.client` for URL fetch on save.
- **Mac App Store:** Set `com.apple.security.app-sandbox` = true and `com.apple.security.network.client` = true so URL fetch on save works.

## Deletion and empty states

- **Deletion:** User can delete items from SavedItemsView. On delete: remove item and cascade to chunks in one transaction. History rows stay; show "Source no longer available" when the source item was deleted.
- **Empty DB:** Show "Save a URL or paste text to build your knowledge base."
- **No search results:** Show "No relevant content found. Try broadening your query."
