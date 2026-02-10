# Embedding model (Core ML) — all-MiniLM-L6-v2

The app uses **sentence-transformers/all-MiniLM-L6-v2** (384 dimensions). Tokenization is done **in Swift** with a WordPiece tokenizer; the Core ML model receives token IDs only.

## Pipeline

- **Input**: `input_ids` (Int32, shape [1, 256]), `attention_mask` (Int32, shape [1, 256])
- **Output**: `embedding` (Float32, shape [1, 384])
- **Post‑processing**: L2-normalize before storing and before search (dot product = cosine similarity)

## Step 0 — What you are building (lock this in)

- Core ML model: input `input_ids`, `attention_mask` → output `embedding` Float32[384]
- Tokenization in Swift (WordPiece), model bundled in app, no runtime downloads
- Deterministic, production-ready, Apple-safe

## Step 1 — Create a Python environment (export only)

Use Python **3.9–3.11** for best Core ML compatibility (Python 3.14 + coremltools 9 may fail conversion).

```bash
python3 -m venv venv
source venv/bin/activate   # or: .\venv\Scripts\activate on Windows
pip install torch transformers coremltools numpy
```

Or use the requirements file:

```bash
pip install -r scripts/requirements-export.txt
```

## Step 2 — Run the export script

From the repo root (with venv activated):

```bash
python scripts/export_embedding_model.py
```

The script will:

1. Download **all-MiniLM-L6-v2** from Hugging Face
2. Export **minilm_vocab.txt** to `KnowledgeCache/Resources/minilm_vocab.txt` (always, even if Core ML conversion fails)
3. Trace the model (mean pooling over BERT output) and convert to Core ML
4. Save **EmbeddingModel.mlmodel** to `KnowledgeCache/EmbeddingModel.mlmodel`

If you see **Core ML conversion failed**, use Python 3.9–3.11 and try again. Alternatively, use a pre-converted Core ML model from the community (e.g. Hugging Face) that matches the I/O above.

## Step 3 — Add the model to Xcode

1. Open the KnowledgeCache Xcode project.
2. Drag **EmbeddingModel.mlmodel** into the project (e.g. into the KnowledgeCache group).
3. Ensure **“Add to target: KnowledgeCache”** is checked.
4. Xcode will compile it to **EmbeddingModel.mlmodelc** at build time.

Do **not** rename inputs/outputs in Xcode. Swift expects:

- Inputs: `input_ids`, `attention_mask`
- Output: `embedding`

## Step 4 — Verify from Swift (optional)

In Swift you can confirm the model description:

```swift
let model = try MLModel(contentsOf: url, configuration: MLModelConfiguration())
print(model.modelDescription)
```

You should see inputs `input_ids`, `attention_mask` and output `embedding` (384).

## What happens at runtime

1. Swift text → **MiniLMTokenizer** → `input_ids` + `attention_mask`
2. **EmbeddingModel.mlmodelc** (Core ML) → Float32[384]
3. L2-normalize → store / search (dot product = cosine similarity)

Fully offline, deterministic, no runtime downloads.

## Versioning and re-index

- Chunks are stored with `embedding_model_id` (e.g. `minilm-l6-v2-v1`) and `embedding_dim` (384).
- If you change the model or dimension, run **Re-index all** from the Saved tab so all chunk embeddings match. Otherwise search shows **Re-index required**.

## Common mistakes to avoid

- Do not try to make Core ML accept a String (tokenize in Swift).
- Do not download models in the app.
- Do not forget mean pooling when exporting from BERT.
- Do not forget L2 normalization when storing/querying.
- Use a fixed max token length (256) in both export and Swift tokenizer.
