# Changesets GitHub Action

This GitHub Action helps you manage the process of releasing changes to your npm packages with [Changesets](https://github.com/atlassian/changesets).

## How to Use

1. Install the package:

   ```bash
   pnpm add -D @changesets/cli @actions/exec

   ```

2. Create the `scripts` folder and add the `release.js` and `bump.js` files:

```js
// scripts/release.js
const path = require("path");
const { exec, getExecOutput } = require("@actions/exec");

const { version } = require("../package.json");

const tag = `v${version}`;
const releaseLine = `v${version.split(".")[0]}`;

process.chdir(path.join(__dirname, ".."));

(async () => {
  const { exitCode, stderr } = await getExecOutput(
    `git`,
    ["ls-remote", "--exit-code", "origin", "--tags", `refs/tags/${tag}`],
    {
      ignoreReturnCode: true,
    }
  );
  if (exitCode === 0) {
    console.log(`Action is not being published because version ${tag} is already published`);
    return;
  }
  if (exitCode !== 2) {
    throw new Error(`git ls-remote exited with ${exitCode}:\n${stderr}`);
  }

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
})();
```

```js
// scripts/bump.js
const fs = require("fs");
const path = require("path");
const { exec } = require("@actions/exec");

process.chdir(path.join(__dirname, ".."));

(async () => {
  await exec("changeset", ["version"]);

  const releaseLine = `v${require("../package.json").version.split(".")[0]}`;

  const readmePath = path.join(__dirname, "..", "README.md");
  const content = fs.readFileSync(readmePath, "utf8");
  const updatedContent = content.replace(
    /changesets\/action@[^\s]+/g,
    `changesets/action@${releaseLine}`
  );
  fs.writeFileSync(readmePath, updatedContent);
})();
```

3. Add the following to `package.json`:

```json
  "scripts": {
    "release-v2": "node scripts/release.js",
    "bump": "node scripts/bump.js",
    "release": "pnpm bump && pnpm release-v2"
  },
```

4. Create file `.github/workflows/release.yml`:

```yml
name: Publish
on:
  workflow_dispatch:

concurrency: ${{ github.workflow }}-${{ github.ref }}

permissions:
  contents: write
  pull-requests: write

jobs:
  publish:
    if: ${{ github.event.workflow_run.conclusion == 'success' }}
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
          publish: pnpm release
        env:
          GITHUB_TOKEN: ${{ secrets.MIKA_TOKEN }}
```

Happy releasing!
