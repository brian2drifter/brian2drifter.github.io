+++
date = '2025-12-11T13:32:03Z'
draft = false
title = 'Hugo started'
+++

# My start with Hugo

It all started with this video (see the main post)

[![alt text](https://img.youtube.com/vi/zrmeOu8DYyw/0.jpg)](https://www.youtube.com/watch?v=zrmeOu8DYyw)

Actually it started a week before I'd even thought of a Hugo blog, but that's a long story best left for another post. Anyway the video was excellent but ... lots of pitfalls on the way.

### Hurdle 1: Installing Hugo on Linux
Surely it's just `sudo apt install hugo`?  Well that works if you want a pretty old version of Hugo, one sufficiently old not to support the Ananke theme, for instance. So download the latest .deb from the Hugo site and all is good perhaps. Well, no, actually! Hugo install the binary to `/usr/local/bin/hugo` and whilst that was in my $PATH Hugo wouldn't run, I assume Go was trying to find Hugo in `usr/bin/hugo`. Anyway nothing a quick copy wouldn't fix.

Still no joy, the ananke theme failed to load - I'd installed Hugo standalone not Hugo extended so didn't have the requisite SASS support. At least a quick reinstall of the fuller version was an easy fix

### Hurdle 2: Git authentication
I'm not really a git user (yes, I know, shame on me) but for projects where I'm a team of 1 then it seems a bit overkill (although I realise that is not really the case). So in my naivety I thought a simple `git credential-manager github login` but on Linux they keys need to be generated manually. So it was:

``` sh
sudo apt install pass gnupg
gpg --full-generate-key
gpg --list-secret-keys --keyid-format=long
pass init *mykey*
git config --global credential.credentialStore
```

and, at last I could connect to github (after sharing my credentialStore passphrase)

### Hurdle 3 - Getting a working build & deploy script
I thought it would be all plain sailing now, just copy the build & deploy gist into Github and away we go. But there were a couple of outdated version settings in the script which caused it to fail. Fixing those and it was still failing. Since this is all a bit new I asked ChatGPT about the error and it decide a whole new gist was the answer

``` yaml
name: Deploy Hugo site to GitHub Pages

on:
  push:
    branches:
      - main
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: false

jobs:
  build:
    runs-on: ubuntu-latest

    env:
      HUGO_VERSION: 0.152.0  # Latest Hugo Extended

    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          submodules: recursive
          fetch-depth: 0

      - name: Install Hugo (Extended)
        run: |
          wget -O ${{ runner.temp }}/hugo.deb \
            https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_linux-amd64.deb
          sudo dpkg -i ${{ runner.temp }}/hugo.deb

      - name: Setup Pages
        id: pages
        uses: actions/configure-pages@v4

      - name: Install Node dependencies (optional)
        run: |
          if [[ -f package-lock.json || -f npm-shrinkwrap.json ]]; then
            npm ci
          fi

      - name: Build with Hugo
        env:
          HUGO_ENVIRONMENT: production
          HUGO_ENV: production
        run: |
          hugo \
            --gc \
            --minify \
            --baseURL "${{ steps.pages.outputs.base_url }}/"

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./public

  deploy:
    runs-on: ubuntu-latest
    needs: build

    environment:
      name: github-pages
      url: ${{ steps.deploy.outputs.page_url }}

    steps:
      - name: Deploy to GitHub Pages
        id: deploy
        uses: actions/deploy-pages@v4
```

At last - a working Hugo blog! A bit more of a slog than expected but straightforward enough eventually.