# Self-hosted Blogpage 

Here is what I did to set up my blog on GitHub Pages, and what it does. 

## Install Dependencies 

Dependencies are programs that our software relies on to function. Just like doing matrix multiplication requires you to know how to multiply a number, our dependencies (specifically libraries) provide the implementations for different functions we run. 

### Git 

Installing git is it's own ordeal. Assuming you are here on GitHub, you should have git installed on your local machine, but if not, there are plenty of other blogs which can help you set things up. To test if you have git installed, you can run `git --version` and it should tell you the version of git installed on your system. 

### Hugo and Go 

To install Hugo and go, use your package manager to install it. On Debian-based systems, the command is:
```Debian 
sudo apt install go hugo 
```
Hugo is an engine which makes it easy to host our websites. We don't really need to know __how__ it works, but we should know what it does. Using a config, we tell Hugo how we want to make the website. 

## Setting up a new project 

To set up a new project, we first can create a directory in our filesystem: 
```
mkdir blogpage && cd blogpage
```
Afterwards, we can create a hugo project inside the directory: 
```
hugo new site .
```
You should also set up a git repository, so that you can keep track of your different versions of the site. This can be done with 
```
git init
git branch -M main
```


## Themes 

Hugo gives you many options in terms of theming. Some are more professional, some are more silly, but you can choose what you like; at the end of the day it doesn't really matter (you can switch themes). 

Themess can be found [here.](https://themes.gohugo.io/)

Once you find a theme that works for you, you can follow the individual documentation in the theme to install it. 

## Blogs 

To add new blogpages, you run: 
```
hugo new posts/hello-world.md
```
or whatever blog you want to make. 

## GitHub Pages integration 

To support GitHub Pages, you should add this to your TOML config: 

```hugo.toml
[caches]
  [caches.images]
    dir = ':cacheDir/images'
```
And create a file, `.github/workflows/hugo.yaml`: 
```
name: Build and deploy
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
  group: pages
  cancel-in-progress: false
defaults:
  run:
    shell: bash
jobs:
  build:
    runs-on: ubuntu-latest
    env:
      DART_SASS_VERSION: 1.99.0
      GO_VERSION: 1.26.1
      HUGO_VERSION: 0.160.0
      NODE_VERSION: 24.14.1
      TZ: Europe/Oslo
    steps:
      - name: Checkout
        uses: actions/checkout@v6
        with:
          submodules: recursive
          fetch-depth: 0
      - name: Setup Go
        uses: actions/setup-go@v6
        with:
          go-version: ${{ env.GO_VERSION }}
          cache: false
      - name: Setup Node.js
        uses: actions/setup-node@v6
        with:
          node-version: ${{ env.NODE_VERSION }}
      - name: Setup Pages
        id: pages
        uses: actions/configure-pages@v6
      - name: Create directory for user-specific executable files
        run: |
          mkdir -p "${HOME}/.local"
      - name: Install Dart Sass
        run: |
          curl -sLJO "https://github.com/sass/dart-sass/releases/download/${DART_SASS_VERSION}/dart-sass-${DART_SASS_VERSION}-linux-x64.tar.gz"
          tar -C "${HOME}/.local" -xf "dart-sass-${DART_SASS_VERSION}-linux-x64.tar.gz"
          rm "dart-sass-${DART_SASS_VERSION}-linux-x64.tar.gz"
          echo "${HOME}/.local/dart-sass" >> "${GITHUB_PATH}"
      - name: Install Hugo
        run: |
          curl -sLJO "https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_linux-amd64.tar.gz"
          mkdir "${HOME}/.local/hugo"
          tar -C "${HOME}/.local/hugo" -xf "hugo_extended_${HUGO_VERSION}_linux-amd64.tar.gz"
          rm "hugo_extended_${HUGO_VERSION}_linux-amd64.tar.gz"
          echo "${HOME}/.local/hugo" >> "${GITHUB_PATH}"
      - name: Verify installations
        run: |
          echo "Dart Sass: $(sass --version)"
          echo "Go: $(go version)"
          echo "Hugo: $(hugo version)"
          echo "Node.js: $(node --version)"
      - name: Install Node.js dependencies
        run: |
          [[ -f package-lock.json || -f npm-shrinkwrap.json ]] && npm ci || true
      - name: Configure Git
        run: |
          git config core.quotepath false
      - name: Cache restore
        id: cache-restore
        uses: actions/cache/restore@v5
        with:
          path: ${{ runner.temp }}/hugo_cache
          key: hugo-${{ github.run_id }}
          restore-keys:
            hugo-
      - name: Build the site
        run: |
          hugo build \
            --gc \
            --minify \
            --baseURL "${{ steps.pages.outputs.base_url }}/" \
            --cacheDir "${{ runner.temp }}/hugo_cache"
      - name: Cache save
        id: cache-save
        uses: actions/cache/save@v5
        with:
          path: ${{ runner.temp }}/hugo_cache
          key: ${{ steps.cache-restore.outputs.cache-primary-key }}
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v5
        with:
          path: ./public
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v5
```

Once you do that, you can git push to your repo and if you have Github Pages enabled, it should work and stuff. 


