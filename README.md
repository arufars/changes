# 🚀 Automated Changelog and Release with Github Release

## 📖 Introduction

This repository provides the configuration for `Automated Changelog and Release with Github Release` using tools like `commitlin`t, `commitizen`, and `release-it` to make software version management and changelog creation easier and automated.

## 🛠️ Installation

### Commitlint + Commitizen

Commitlint and Commitizen are tools used to ensure that commits are made according to the specified rules. You can see the rules in the `commitlint.config.js` and `commitizen.config.js` files.

### Commitlint

You can install Commitlint by running the following command:

```bash
npm install --save-dev @commitlint/config-conventional @commitlint/cli
```

### Commitizen

install Commitizen by running the following command:

```bash
npm install --save-dev commitizen @commitlint/cz-commitlint
```

### Release-it

Release-it is a tool used to release your project automatically or manually. We use the `@release-it/conventional-changelog` plugin to create changelogs automatically. You can install it by running the following command:

```bash
npm install --save-dev release-it @release-it/conventional-changelog
```

## 📝 Konfigurasi

### Commitlint + Commitizen

First, create a `commitlint.config.js` file at the root of the project and fill it with the following configuration:

```js
module.exports = { extends: ["@commitlint/config-conventional"] };
```

Next, add the configuration to the package.json as follows:

```json
 "config": {
    "commitizen": {
      "path": "@commitlint/cz-commitlint"
    }
  }
```

### Release-it

First, create a release-it.json file at the root of the project and fill it with the following configuration:

```json
 "release-it": {
    "git": {
      "changelog": "git fetch --all && git log --pretty=format:\"* %s (%h)\" `git describe --tags --abbrev=0`..HEAD",
      "commitMessage": "v${version}"
    },
    "github": {
      "release": true,
      "releaseName": "v${version}",
      "tokenRef": "GITHUB_TOKENS"
    },
    "npm": {
      "publish": false
    },
    "plugins": {
      "@release-it/conventional-changelog": {
        "infile": "CHANGELOG.md",
        "preset": {
          "name": "conventionalcommits",
          "types": [
            {
              "type": "feat",
              "section": "### ✨ Features"
            },
            {
              "type": "fix",
              "section": "### 🐛 Bug Fixes"
            },
            {
              "type": "chore",
              "hidden": true
            },
            {
              "type": "docs",
              "hidden": true
            },
            {
              "type": "style",
              "hidden": true
            },
            {
              "type": "refactor",
              "hidden": true
            },
            {
              "type": "perf",
              "hidden": true
            },
            {
              "type": "test",
              "hidden": true
            }
          ],
          "releaseCount": 1,
          "outputUnreleased": true,
          "whoCanContribute": "CONTRIBUTORS.md"
        },
        "contributors": true
      }
    }
  }
```

### Script Package.json

Add the script to `package.json` as follows:

```json
{
  "scripts": {
    "commit": "git-cz",
    "release": "dotenv release-it --",
    "release:beta": "dotenv release-it -- --preRelease=beta"
  }
}
```

### Create environment variable

> You can get your github token [here]()

```bash
GITHUB_TOKENS=your_github_token
```

### Create .env file

```bash

touch .env

```

### Another way use `dotenv-cli`

```json
{
  "script": {
    "release": "set /p GITHUB_TOKEN=Enter your GitHub token: && set \"GITHUB_TOKEN=%GITHUB_TOKEN%\" && release-it"
  }
}
```

## 👉 Usage

### Commit

```bash

npm run commit

```

### Release

```bash

npm run release

```

### 📝 Referensi

- [Commitlint]()
- [Commitizen]()
- [Release-it]()
- [Release-it Conventional Changelog]()

### 📝 Lisensi

MIT

### 📝 Author

[Arufars 👨‍💻]()
