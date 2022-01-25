# anisimov.dk blog

## Developing

Run jekyll locally

```bash
docker run -it --rm -v "$PWD":/src/site -p "4000:4000" ghcr.io/github/pages-gem
```

Run jekyll in production mode locally

```bash
docker run -it --rm -v "$PWD":/src/site -p "4000:4000" -e JEKYLL_ENV=production ghcr.io/github/pages-gem
```
