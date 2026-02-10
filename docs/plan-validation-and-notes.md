# Offline Knowledge Memory MVP — Plan Validation & Implementation Notes

**Document version:** 1.0  
**Date:** February 9, 2025  
**Source:** Plan validation for offline_knowledge_memory_mvp_4ab3db45.plan.md

## 1. Executive Summary

The Offline Knowledge Memory MVP plan is technically sound and implementation-ready. This document records validation feedback, recommendations, and implementation notes from two review passes. The main risk is underestimating the Core ML embedding pipeline; validating it early is strongly recommended.

## 2. Strengths of the Plan

| Strength | Description |
|----------|-------------|
| Offline-first scope | Clear: network used only for URL fetch during save |
| Pipeline separation | Ingestion and query flows are explicit and testable |
| Schema versioning | schema_version and migrations planned from the start |
| Shipping focus | Signing, notarization, export/backup, sandbox considered |
| History | Query history fits "memory" concept and enables future analysis |
| Timestamps | created_at on items and queries supports auditing and date-range queries |

## 3. Critical Path Items (Must Address)

### 3.1 Embedding Model Pipeline — Highest Risk

**Issue:** Conversion and integration of a sentence-transformers model to Core ML is non-trivial.

**Required checks:**
- **Input format:** Models expect tokenized input (input_ids, attention_mask). Core ML conversion must match; you may need a tokenizer (e.g., Hugging Face) or a model that includes tokenization.
- **Output format:** Ensure the model outputs a single vector per input (not per-token).
- **Dimension:** 384 is typical; do not hardcode; read from the model at runtime.

**Action:** Pick one model (e.g., all-MiniLM-L6-v2), run the full export → load → embed flow in a minimal Swift test app before building the rest.

### 3.2 Chunking and Token Limits

**Issue:** "512 tokens or ~2000 chars" is underspecified. Character-based chunking can exceed model token limits.

**Recommendation:** Use character-based chunking with a conservative cap (e.g., 500–600 chars for English) to stay under 512 tokens. Or add a tokenizer and chunk by tokens. Avoid "~2000 chars" without verification.

**Suggested parameters:**
- `maxChars: 600`
- `overlapChars: 100`
- Split on paragraph/sentence boundaries when possible

### 3.3 BLOB Format for Embeddings

**Issue:** Plan says "BLOB of floats" but does not specify layout.

**Specification:**
- **Format:** Float32 (4 bytes per value)
- **Layout:** 384 dims × 4 bytes = 1,536 bytes per embedding
- **Byte order:** Native (little-endian on macOS)

Store this in a `docs/embedding-schema.md` or schema comment.

## 4. Important Recommendations

### 4.1 Async / Background Processing

**Issue:** Plan says "single-thread is fine." Embedding and retrieval can take seconds and will freeze the UI if on the main thread.

**Recommendation:** Run the ingestion pipeline (fetch → extract → chunk → embed → store) on a background queue. Same for query pipeline if it may exceed ~200–300 ms. Use @MainActor for UI updates. Add progress feedback (e.g., "Embedding 3 of 12 chunks…").

### 4.2 Implementation Order

**Current order:** Schema → Models → TextExtractor → Chunker → Embedding → Pipeline → …

**Issue:** Embedding is the highest-risk dependency. Discovering problems late forces rework.

**Recommendation:** Validate the embedding pipeline right after project setup (step 2):
1. Project setup
2. **Embedding:** integrate Core ML model + EmbeddingService, verify it works
3. Schema + Models
4. TextExtractor, Chunker
5. Rest of pipeline

### 4.3 TextExtractor Implementation

**Options:**
- **Reeeed** (SPM): URL → extracted content
- **swift-readability:** Mozilla Readability port
- **Simple:** Fetch HTML, strip tags, basic heuristics

**Note:** Reader-mode works on static HTML. JavaScript-rendered content will not be extracted. Treat this as a known limitation.

### 4.4 AnswerGenerator MVP Behavior

**Plan options:** (A) ranked chunks as-is, (B) concatenate with section headers.

**Recommendation:** Use (B) for better UX; extra work is small. Format: headers per chunk, "Source: [title/URL]" labels.

### 4.5 Deletion Semantics

**Plan:** SavedItemsView lists items; deletion is implied but not specified.

**Recommendation:** Allow deletion from SavedItemsView. On delete: remove item and associated chunks in one transaction. For history: keep rows, but show "Source no longer available" when the source item was deleted.

## 5. Schema & Data Model Notes

### 5.1 History Sources Storage

**MVP:** Use a JSON blob of [SourceRef] in query_history. Keeps schema simple and matches AnswerWithSources. If needed later, add a history_sources table for analytics.

### 5.2 SourceRef for Pasted Text

For pasted items, url is nil. SourceRef must support:
- title: first line or "Pasted text"
- snippet: chunk excerpt
- url: optional, nil for paste

Ensure AnswerWithSources and UI handle both cases without excessive branching.

### 5.3 Migration Flow

- On launch, read `PRAGMA user_version`.
- If version < target, run migrations in order (v1→v2, v2→v3, …).
- After success, set `PRAGMA user_version` to target.
- Wrap each migration in a transaction for rollback on failure.

## 6. UX & Edge Cases

| Case | Recommendation |
|------|----------------|
| Empty DB (first run) | Show "Save a URL or paste text to build your knowledge base" |
| No search results | Show "No relevant content found. Try broadening your query." |
| Duplicate URLs | Allow duplicates in MVP; add deduplication later if desired |
| Long documents | Progress feedback (e.g., "Embedding 3/10 chunks…") |
| Export | Clarify: SQLite copy = full backup; JSON = inspection/migration only |

## 7. Technical Choices to Lock In

| Decision | Options | Recommendation |
|----------|---------|----------------|
| Embedding model | 384-dim sentence transformer | all-MiniLM-L6-v2 (Hugging Face exporters / coremltools) |
| SQLite | SQLite.swift vs raw sqlite3 | Raw sqlite3 for minimal deps; SQLite.swift acceptable for ergonomics |
| Similarity metric | Cosine vs dot product | Dot product if embeddings are L2-normalized; otherwise cosine |
| Minimum macOS | Not specified | macOS 13+ (or 14+ if the chosen model requires it) |
| Entitlement | Network for URL fetch | com.apple.security.network.client when sandboxed |

## 8. Pre-Implementation Checklist

- [ ] Embedding model chosen and conversion path documented
- [ ] Minimal embedding test app passes (load model, embed strings)
- [ ] Chunking parameters fixed (e.g., 600 chars, 100 overlap)
- [ ] BLOB format documented
- [ ] TextExtractor approach chosen (library or simple heuristic)
- [ ] Minimum macOS version set
- [ ] Async/background strategy for ingestion and search defined

## 9. Shipping Checklist (from Plan)

- [ ] Schema version checked on startup; migrations run when needed
- [ ] Export or backup implemented
- [ ] App signed with Developer ID, notarized, and stapled
- [ ] Install tested on a clean Mac; no Gatekeeper block
- [ ] URL save works with network; app works fully offline after save

— End of document
