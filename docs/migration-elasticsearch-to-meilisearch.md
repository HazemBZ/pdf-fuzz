# Migration: Elasticsearch → Meilisearch

## Rationale

Meilisearch is significantly lighter (256-512 MB RAM vs 2-4 GB for ES), runs as a single Rust binary (no JVM), and fits free-tier cloud services. The current ES dependency makes cloud demo hosting impractical.

## Stack impact

| Layer | Before | After |
|-------|--------|-------|
| Search engine | Elasticsearch 8.14 | Meilisearch latest |
| Python client | `elasticsearch` | `meilisearch` |
| Docker image | `docker.elastic.co/elasticsearch/elasticsearch:8.14.0` | `getmeili/meilisearch:latest` |
| Port | 9200 | 7700 |
| Env vars | `ELASTIC_ADDRESS` | `MEILI_ADDR`, `MEILI_MASTER_KEY` |

## Document schema

Document structure stays the same; adds a `page_doc_id` primary key:

```json
{
  "page_doc_id": "{upload_id}_{page_id}",
  "upload_id": "uuid",
  "file_path": "document.pdf",
  "content": "full page text...",
  "page_number": 1,
  "page_id": 1
}
```

## API mapping

| ES method | Meilisearch equivalent |
|-----------|----------------------|
| `es.bulk(operations)` | `index.add_documents(documents)` |
| `es.search(query={match: {content: kw}})` | `index.search(keyword, {attributesToRetrieve})` |
| `es.indices.delete()` / `.create()` | `index.delete()` (auto-recreated on next add) |
| `es.delete_by_query(...)` | `index.delete_documents(ids)` or `index.delete_all_documents()` |
| `es.info()` | `client.health()` |
| `es.search(query={match_all: {}})` | `index.get_documents()` |

## Files to modify

| # | File | Change |
|---|------|--------|
| 1 | `pyproject.toml` | `elasticsearch==8.14.0` → `meilisearch` |
| 2 | `requirements.txt` | Same dep swap |
| 3 | `docker-compose.yaml` | Replace `elastic` service with `meilisearch`; update env vars for backend/celery/dashboard |
| 4 | `fuzz/search.py` | Rewrite `Search` class with Meilisearch `Client` |
| 5 | `fuzz/views.py` | Flatten result access (`_source` → direct) |
| 6 | `fuzz/etl/PdfETL.py` | `es_index` → `index` |
| 7 | `PDF_Fuzz/__init__.py` | Update env var name for connection |
| 8 | `fuzz/management/commands/cleanup.py` | `Search.create_index()` → `Search.reset_index()` |
| 9 | `fuzz/management/commands/alldocs.py` | Result access pattern |
| 10 | `fuzz/management/commands/testquery.py` | Result access pattern |
| 11 | `fuzz/tests/test_pdf_etl.py` | `es_manager` → `meili_manager`, update search call |

Files with **no changes**: frontend (`pdf-fuzz-front/`) — API contract and response shape are unchanged.

## Docker compose changes

```yaml
meilisearch:
  image: getmeili/meilisearch:latest
  container_name: meili_search
  ports:
    - "7700:7700"
  environment:
    MEILI_MASTER_KEY: "dev-master-key"
    MEILI_NO_ANALYTICS: "true"
  volumes:
    - meili_data:/meili_data
  healthcheck:
    test: ["CMD", "curl", "-f", "http://localhost:7700/health"]
```

Update backend/celery env:
- `ELASTIC_ADDRESS=elastic` → `MEILI_ADDR=http://meilisearch:7700` + `MEILI_MASTER_KEY=dev-master-key`

## Search result format change

**ES response** (current):
```python
item["_source"]["file_path"]  # nested under _source
```

**Meilisearch response** (new):
```python
item["file_path"]  # flat, top-level
```

## Test changes

Fixture `es_manager` → `meili_manager`; cleanup uses `delete_all_documents()` instead of `delete_by_query`.

## Notes

- `reindex.py` management command stays dead (was already commented out)
- Meilisearch is auto-schema — no explicit mapping definition needed
- Primary key (`page_doc_id`) is added inside `Search.insert_documents()` so callers don't change
