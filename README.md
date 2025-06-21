# O-Week

This is a O-Week tutorial site built with [MkDocs Material](https://squidfunk.github.io/mkdocs-material/).

📖 Live Site: https://hrc-pme.github.io/O-Week/

## Local Preview (with Docker)

```bash
docker run --rm -it -p 8000:8000 -v $(pwd):/docs squidfunk/mkdocs-material
```

## Tools Used

- **MkDocs**: A fast and simple static site generator using Markdown.
- **Material for MkDocs**: A clean and powerful theme with many useful features.
- **Docker**: The entire development and build process can be done inside containers — no need to install Python packages locally.