# LangExtract: Unstructured → Structured

The app can optionally convert raw extracted page text into **structured** form (summary, key points, facts) using [LangExtract](https://github.com/google/langextract) before chunking and indexing.

## How it works

1. **Script**: `scripts/extract_structured.py` uses the LangExtract Python library to run an LLM over the raw body text and extract:
   - **Summary** – short overview
   - **Key points** – bullet-style points
   - **Facts** – important entities/facts

2. **Swift**: When saving a URL, if the script is present in the app bundle, the pipeline runs it (via `StructuredExtractor`), passes the raw text on stdin, and uses the script’s stdout as the content to chunk and store. If the script is missing or fails (e.g. no API key), the app falls back to the original unstructured text.

3. **Result**: Indexed content is cleaner and more query-friendly (summary + bullets + facts instead of raw HTML-like text).

## Setup

1. **Install LangExtract** (Python 3.10+):

   ```bash
   cd scripts
   pip install -r requirements-langextract.txt
   ```

2. **API key** (for cloud models):

   - **Gemini** (default in script): set `LANGEXTRACT_API_KEY` or `GOOGLE_API_KEY`.
   - **Ollama** (local): no key; install [Ollama](https://ollama.com), run `ollama pull gemma2:2b`, and ensure `ollama serve` is running. The script will use `gemma2:2b` if no API key is set.

3. **App**: The script is bundled in the app (Copy Resources). Ensure `scripts/extract_structured.py` is added to the app target resources so `Bundle.main.url(forResource: "extract_structured", withExtension: "py")` finds it. Then the pipeline will use it automatically when saving URLs.

## Disabling

Remove or don’t add `extract_structured.py` to the app’s Copy Resources so the pipeline never receives a script URL and always uses raw text.
