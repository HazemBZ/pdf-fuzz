# pdf-fuzz

PoC bulk search your pdf files using fuzzy text look up.

## How to


__Requirements__

- Docker
- docker-compose

__Run this project__

1.Clone project and submodules: run `git clone --recurse-submodules https://github.com/HazemBZ/pdf-fuzz`.

2.Spin up containers: run `docker-compose up`.

3.Access app @`http://localhost:88`

## TODOs 

__V0-PoC__

- [x] CI/CD: docker-compose -> one click project spin up.
- [x] BE: ETL solution for text lookup -> Faster lookups, extract once use forever.
- [x] FE: Handle queries w/ ReactQuery -> DX.
- [x] FE/BE: Files uploader -> QoL.


__V1__

- [x] BE: Task Queue solution for files processing -> Seperation of concerns.
- [x] FE/FE: Basic file deduplication
- [x] BE: Refactor into pipelines and orchestrators
- [x] BE: Test code
- [x] BE: (docs) Add diagrams
- [x] CI/CD: Auto migration setup
