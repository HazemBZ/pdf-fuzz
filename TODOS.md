# Migration: Elasticsearch → Meilisearch

- [ ] **1. Swap dependencies** — `pyproject.toml` & `requirements.txt`: remove `elasticsearch==8.14.0`, add `meilisearch`
- [ ] **2. Update docker-compose** — replace `elastic` service with `meilisearch`, update env vars for backend/celery/dashboard, add healthcheck, update volumes
- [ ] **3. Rewrite `fuzz/search.py`** — replace `Elasticsearch` client with Meilisearch `Client`, add `page_doc_id` primary key, map all methods
- [ ] **4. Update `fuzz/views.py`** — flatten result access from `_source` pattern to direct field access
- [ ] **5. Update `fuzz/etl/PdfETL.py`** — rename `es_index` to `index`
- [ ] **6. Update `PDF_Fuzz/__init__.py`** — change env var from `ELASTIC_ADDRESS` to `MEILI_ADDR`
- [ ] **7. Update `cleanup.py`** — replace `Search.create_index()` with `Search.reset_index()`
- [ ] **8. Update `alldocs.py`** — fix result access pattern for Meilisearch format
- [ ] **9. Update `testquery.py`** — fix result access pattern for Meilisearch format
- [ ] **10. Update `test_pdf_etl.py`** — rename fixture `es_manager` → `meili_manager`, update search/cleanup calls
- [ ] **11. Run tests** — `pytest` to verify everything works end-to-end
- [ ] **12. Run lint** — `flake8` / `ruff` to check code quality
