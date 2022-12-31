# Zola Build Action
This GitHub Action enables you to build your [Zola](https://www.getzola.org/) site.

This repository is a fork of [Tony Spegel's repository](https://github.com/TonySpegel/zola-build-action).

## Usage
The following example builds your Zola website using the main branch on push. There are two [environment variables](#environment-variables) available.

By specifying your version with `Hysterelius/zola-build-action@vx.x.x` you can specify which version of [Zola](https://github.com/getzola/zola) to use in the build step. 

(This tool only supports [Zola](https://github.com/getzola/zola) version 0.13.0 and onwards)

This build tool outputs to the `dist` folder for easy deployment to GitHub pages or Cloudflare pages.
```yml
# .github/workflow/main.yml
on: 
  push:
    branches:
      - main
name: Build a Zola website
jobs:
  build-bundle-deploy:
    runs-on: ubuntu-latest
    steps:
      # Check-out repository under $GITHUB_WORKSPACE
      - name: Checkout 📋
        uses: actions/checkout@v3

      # Build static pages w/ Zola
      - name: Build 🏗️
        uses: Hysterelius/zola-build-action@v0.16.1
        env:
          CONFIG_FILE: "config.toml"
```
## Deployment

### GitHub pages
You can use this workflow to build then deploy your site to [GitHub pages](https://pages.github.com/).
```yml
# .github/workflow/main.yml
on: 
  push:
    branches:
      - main
name: Build, Bundle & Deploy
jobs:
  build-bundle-deploy:
    runs-on: ubuntu-latest
    steps:
      # Check-out repository under $GITHUB_WORKSPACE
      - name: Checkout 📋
        uses: actions/checkout@v3

      # Build static pages w/ Zola
      - name: Build 🏗️
        uses: Hysterelius/zola-build-action@v0.16.1
        env:
          CONFIG_FILE: "config.toml"

      # Deploy to GitHub Pages
      - name: Deploy 🚀
        uses: JamesIves/github-pages-deploy-action@4.4.1
        with:
          branch: gh-pages # The branch the action should deploy to.
          folder: dist # The folder the action should deploy.          
```
It is also possible to bundle your website with Rollup, to do that I would highly recommend looking at the original creator's [repository](https://github.com/TonySpegel/tsp-website) with
```yml
# Install NPM dependencies & bundle w/ Rollup
      - name: Bundle 🧶
        uses: actions/setup-node@v2
        with:
          node-version: '16'
      - run: npm install
      - run: sudo npm run build-rollup-live
```

### Cloudflare Pages
Or instead you can deploy your Zola website to [Cloudflare pages](https://pages.cloudflare.com/) instead just after the build.

```yml
# .github/workflow/main.yml
on: 
  push:
    branches:
      - main
name: Build, Bundle & Deploy
jobs:
  build-bundle-deploy:
    runs-on: ubuntu-latest
    steps:
      # Check-out repository under $GITHUB_WORKSPACE
      - name: Checkout 📋
        uses: actions/checkout@v3

      # Build static pages w/ Zola
      - name: Build 🏗️
        uses: Hysterelius/zola-build-action@v0.16.1
        env:
          CONFIG_FILE: "config.toml"

      # Deploy to Cloudflare Pages
      - name: Deploy 🚀
        uses: cloudflare/pages-action@v1
        with:
          apiToken: ${{ secrets.CLOUDFLARE_API_TOKEN }}
          accountId: YOUR_ACCOUNT_ID
          projectName: YOUR_PROJECT_NAME
          directory: YOUR_ASSET_DIRECTORY
          # Optional: Enable this if you want to have GitHub Deployments triggered
          gitHubToken: ${{ secrets.GITHUB_TOKEN }}      
```

The program is mostly a shellscript:
```bash
#!/bin/bash

# [ -z STRING ]: True if the length of "STRING" is zero.

set -e
set -o pipefail

if [[ -z "$BUILD_DIR" ]]; then
    BUILD_DIR="."
fi

if [[ -z "$CONFIG_FILE" ]]; then
    CONFIG_FILE="config.toml"
fi


main() {
    version=$(zola --version)

    echo "Using $version"

    echo "Building in $BUILD_DIR directory"
    cd $BUILD_DIR

    zola --config ${CONFIG_FILE} build --output-dir dist
    chmod -R 777 dist
}

main "$@"

```
The Docker file downloads and installs the Zola binary and runs the shellscript.

## Environment variables
- `BUILD_DIR`: Where to run `zola build` from. Default is `.` (the current directory)
- `CONFIG_FILE`: Which config file to use to build with Zola. Default is `config.toml`
