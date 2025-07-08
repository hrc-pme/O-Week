# NTHU HRC Tech Docs

> Welcome to the official documentation site for the NTHU HRC, powered by [MkDocs Material](https://squidfunk.github.io/mkdocs-material/).

**Live Documentation:** [https://hrc-pme.github.io/hrc-tech-docs/](https://hrc-pme.github.io/hrc-tech-docs/)

</br>

## Local Dev Preview (Docker)

To preview the documentation locally using Docker, run `source run.sh` or 

```bash
docker run --rm -it -p 8000:8000 -v $(pwd):/docs hrcnthu/mkdocs:default
```