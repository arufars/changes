# Changesets GitHub Action

This GitHub Action helps you manage the process of releasing changes to your npm packages with [Changesets](https://github.com/atlassian/changesets).

## How to Use

1. Install the package:

```bash
pnpm add -D @changesets/cli @actions/exec

```

2. Create the `scripts` folder and add the `release.js` and `version.js` files:

```js
// scripts/release.js
const path = require("path");
const { exec, getExecOutput } = require("@actions/exec");

const { version } = require("../package.json");

const tag = `v${version}`;
const releaseLine = `v${version.split(".")[0]}`;

async function checkVersionPublished() {
  try {
    process.chdir(path.join(__dirname, ".."));
    const { exitCode, stderr } = await getExecOutput(
      `git`,
      ["ls-remote", "--exit-code", "origin", "--tags", `refs/tags/${tag}`],
      {
        ignoreReturnCode: true,
      }
    );
    if (exitCode === 0) {
      console.log(`Action is not being published because version ${tag} is already published`);
      return true;
    }
    if (exitCode !== 2) {
      throw new Error(`git ls-remote exited with ${exitCode}:\n${stderr}`);
    }
    return false;
  } catch (error) {
    console.error(error);
    return false;
  }
}

async function publish() {
  try {
    process.chdir(path.join(__dirname, ".."));
    await exec("git", ["checkout", "--detach"]);
    await exec("git", ["add", "--force", "dist"]);
    await exec("git", ["commit", "-m", tag]);
    await exec("changeset", ["tag"]);
    await exec("git", [
      "push",
      "--force",
      "--follow-tags",
      "origin",
      `HEAD:refs/heads/${releaseLine}`,
    ]);
  } catch (error) {
    console.error(error);
  }
}

(async () => {
  if (!(await checkVersionPublished())) {
    await publish();
  }
})();

```

```js
// scripts/version.js
const fs = require("fs");
const path = require("path");
const { exec } = require("@actions/exec");

async function updateReadme() {
  try {
    const { version } = require("../package.json");
    const releaseLine = `v${version.split(".")[0]}`;
    const readmePath = path.join(__dirname, "..", "README.md");
    const content = fs.readFileSync(readmePath, "utf8");
    const updatedContent = content.replace(
      /changesets\/action@[^\s]+/g,
      `changesets/action@${releaseLine}`
    );
    fs.writeFileSync(readmePath, updatedContent);
  } catch (error) {
    console.error(error);
  }
}

async function updateVersion() {
  try {
    process.chdir(path.join(__dirname, ".."));
    await exec("changeset", ["version"]);
  } catch (error) {
    console.error(error);
  }
}

(async () => {
  await updateVersion();
  await updateReadme();
})();

```

3. Add the following to `package.json`:
```json
  "scripts": {
    "release": "node scripts/release.js",
    "version": "node scripts/version.js",
    "publish": "pnpm version && pnpm release"
  },
```

4. Create file `.github/workflows/release.yml`:

```yml
name: release
on:
  push:
    branches:
      - "main"

concurrency: ${{ github.workflow }}-${{ github.ref }}

permissions:
  contents: write
  pull-requests: write

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: pnpm/action-setup@v2
        with:
          version: 7
      - uses: actions/setup-node@v3
        with:
          node-version: 16.x
          cache: "pnpm"

      - run: pnpm install --frozen-lockfile

      - name: Build
        run: pnpm build

      - name: Create Release Pull Request or Publish
        id: changesets
        uses: changesets/action@v1
        with:
          publish: pnpm publish
        env:
          GITHUB_TOKEN: ${{ secrets.MIKA_TOKEN }}
```

5. Create file `.github/workflows/publish-package.yml`:
```yml
name: Publish package nodejs

on:
  workflow_run:
    workflows: ["release"]
    types: [completed]

jobs:
  publish:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: pnpm/action-setup@v2
        with:
          version: 7
      - uses: actions/setup-node@v2
        with:
          node-version: "16.x"
          registry-url: "https://registry.npmjs.org/"
          cache: "pnpm"
      - run: pnpm install --frozen-lockfile
      - run: pnpm run build
      - run: npm publish --access=public
        env:
          NODE_AUTH_TOKEN: ${{ secrets.NPM_SECRET }}

```

6. CI `github/workflows/main.yml`:

```yml
name: CI
on:
  push:
    branches:
      - "main"

concurrency: ${{ github.workflow }}-${{ github.ref }}

permissions:
  contents: write
  pull-requests: write

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: pnpm/action-setup@v2
        with:
          version: 7
      - uses: actions/setup-node@v3
        with:
          node-version: 16.x
          cache: "pnpm"

      - run: pnpm install --frozen-lockfile
      - run: pnpm run lint

```

Happy releasing!
