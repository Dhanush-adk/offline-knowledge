# Embedding schema

## BLOB format (chunks.embedding_blob)

- **Type:** `Float32` (4 bytes per value).
- **Layout:** Contiguous array, 384 dimensions × 4 bytes = **1,536 bytes** per embedding.
- **Byte order:** Native (little-endian on macOS).
- **Normalization:** Embeddings are **L2-normalized** before storage and when embedding the query. Similarity is **dot product** (equivalent to cosine for normalized vectors).

## Reading/writing in Swift

- **To BLOB:** `floats.withUnsafeBufferPointer { Data(buffer: $0) }`.
- **From BLOB:** `data.withUnsafeBytes { Array($0.bindMemory(to: Float.self)) }`.
