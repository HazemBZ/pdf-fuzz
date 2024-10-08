# pdf-fuzz

PoC bulk search your pdf files using fuzzy text look up.

## How to


__Requirements__

- Docker
- docker-compose

__Run this project__

1.Clone project and submodules: run `git clone --recurse-submodules https://github.com/HazemBZ/pdf-fuzz`.

2.Drop a folder with pdf files inside `pdf_fuzz_back/assets` folder (smaller number of files -> less time to process).

3.Index db with pdf contents: `docker-compose exec backend bash -c "python manage.py reindex"`.

4.Spin up containers: run `docker-compose up`.


__Update your pdf file__

After changing the contents of `pdf_fuzz_back/assets`, reindex with: `docker-compose exec backend bash -c "python manage.py reindex"`.



## TODOs 

- [x] v0 PoC.
- [x] CI/CD: docker-compose -> one click project spin up.
- [x] BE: ETL solution for text lookup -> Faster lookups, extract once use forever.
- [ ] FE: Handle queries w/ ReactQuery -> DX.
- [ ] FE/BE: Files uploader -> QoL.
- [ ] BE: Task Queue solution for files processing -> Seperation of concerns.