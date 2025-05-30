# pdf-fuzz

PoC bulk search your pdf files using fuzzy text look up.

## How to


__Requirements__

- Docker
- docker-compose

__Run this project__

1.Clone project and submodules: run `git clone --recurse-submodules https://github.com/HazemBZ/pdf-fuzz`.

2.Spin up containers: run `docker-compose up`.

3.First time setup: `docker-compose exec backend bash -c "./manage.py makemigrations; ./manage.py migrate"`


__Update your pdf file__

After changing the contents of `pdf_fuzz_back/assets`, reindex with: `docker-compose exec backend bash -c "python manage.py reindex"`.



## TODOs 

- [x] v0 PoC.
- [x] CI/CD: docker-compose -> one click project spin up.
- [x] BE: ETL solution for text lookup -> Faster lookups, extract once use forever.
- [x] FE: Handle queries w/ ReactQuery -> DX.
- [x] FE/BE: Files uploader -> QoL.
- [x] FE/FE: Basic file deduplication


__V1__

- [ ] BE: Task Queue solution for files processing -> Seperation of concerns.