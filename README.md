# pdf-fuzz

PoC  bulk search you pdf files using text look up.

## How to

__Run this project__

1.Clone project and submodules: run `git clone --recurse-submodules https://github.com/HazemBZ/pdf-fuzz`.

2.Drop a folder with pdf files inside `pdf_fuzz_back/assets` folder (smaller number of files -> less time to process).

3.Spin up containers: run `docker-compose up`.


__Update your pdf file__

After changing the contents of `pdf_fuzz_back/assets`, run: `docker-compose up --build` and that will copy new content inside the container.



## TODOs 

- [x] v0 PoC.
- [ ] FE: Handle queries w/ ReactQuery -> DX.
- [ ] CI/CD: docker-compose -> one click project spin up.
- [ ] FE/BE: Files uploader -> QoL.
- [ ] BE: Task Queue solution for files processing -> Seperation of concerns.
- [ ] BE: ETL solution for text lookup -> Faster lookups, extract once use forever.