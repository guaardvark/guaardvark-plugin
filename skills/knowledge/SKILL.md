---
name: knowledge
description: >-
  Answer from the user's own indexed documents with Guaardvark's local RAG, browse and
  read indexed files section by section, remember facts across sessions, and process or
  fetch new material. Use when the user asks what their documents say, wants a
  source-cited answer from their files, says "remember this", or asks to index a PDF,
  DOCX, spreadsheet, image or URL.
---

# Knowledge with Guaardvark (MCP tools)

Everything here is the `guaardvark` MCP server. All retrieval is local (Ollama embeddings,
hybrid search, cross-encoder reranking on the user's GPU).

## Find and read

| tool | use |
|---|---|
| `search_knowledge_base` | the question in plain words; returns passages with sources. Cite them. |
| `summarize_corpus` | what the whole knowledge base covers, before searching blind |
| `list_documents` | which documents are indexed and how many passages each has |
| `get_document_outline` | sections or pages of one document, in order |
| `read_document_section` | the actual text of one section or page, no search |
| `process_file` | extract content from a PDF, DOCX, CSV, Excel, image or other file the user names |
| `index_document` | add a file in Guaardvark's uploads, outputs or install folder to the knowledge base, so the tools above find it |
| `fetch_url` | title, description and main text of one web page; pass `query` (the user's question) to get the passage about it instead of the top of the page |
| `web_search` | web results (titles, snippets, URLs) when the answer is not local; `source` names the search engine that answered, and `no_results` says when it found nothing |

Pattern: `summarize_corpus` or `list_documents` → `search_knowledge_base` → read the exact
section with `get_document_outline` + `read_document_section` before quoting.

## Remember

- `save_memory`: a fact, preference or instruction to keep across sessions. Use when the user says
  "remember", or states a durable preference. One fact per call.
- `search_memory`: check before assuming; recalled memories are what was true when saved.
- `update_memory`: correct a saved fact, preference or note in place, by the ID `search_memory`
  prints, instead of saving a second version next to it.
- `delete_memory`: forget one entry by its ID when the user asks. It cannot be undone.
- Over MCP these two change only fact, preference and note entries; lessons and Guaardvark's own
  entries are managed on the Agent Memory page.

## Rules

- Quote the source (document name and section) for any claim that came from retrieval.
- If retrieval returns nothing, say so; do not fill the gap from general knowledge without
  labelling it.
- `index_document` adds one file at a time and waits up to `wait_seconds` (default 45) for the
  indexer; a long PDF may still be indexing when it answers, and `list_documents` shows it once
  done. Whole folders are indexed from the Studio's Library page.
