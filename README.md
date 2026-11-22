# Adrien Poupa Blog

This is the code of my blog, built using Hugo and the [PaperMod theme](https://github.com/adityatelange/hugo-PaperMod/).

See https://adrien.poupa.net

## Deployment

Production builds run on Cloudflare Pages. PaperMod requires Hugo >= 0.158, which is newer than the Pages build image default, so set the build environment variable `HUGO_VERSION` to `0.170.1` for both the Production and Preview environments in the Cloudflare Pages dashboard. `docker-compose.yml` only controls the local development server.