# setup-node-action GitHub action

Shared environment setup for Diplodoc workflows. It removes the boilerplate that
was previously copy-pasted into every workflow across the metapackage:

1. Configures Node.js via [`actions/setup-node`](https://github.com/actions/setup-node).
2. Bumps npm to the required version using a fork-safe procedure (user-writable
   prefix + `$GITHUB_PATH`), only when the runner ships an older npm.
3. Optionally runs `npm ci`.

`actions/checkout` must run **before** this action.

## Inputs

- `node-version` (default: `""`) - Node.js version. Usually `${{ vars.NODE_VERSION }}`.
- `npm-version` (default: `11.5.1`) - Minimum required npm version. npm is upgraded only if the runner has an older one.
- `registry-url` (default: `""`) - npm registry URL. Needed for publish / dependency-update workflows.
- `cache` (default: `npm`) - Package manager cache for `actions/setup-node`. Set to `""` to disable.
- `run-install` (default: `true`) - Run `npm ci` after setup. Set to `false` when the install step must be conditional or customized.

## Usage

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - uses: diplodoc-platform/setup-node-action@v1
        with:
          node-version: ${{ vars.NODE_VERSION }}

      - run: npm test
```

When the install step must stay conditional (e.g. in release workflows), disable
the built-in install and run it yourself:

```yaml
      - uses: diplodoc-platform/setup-node-action@v1
        with:
          node-version: ${{ vars.NODE_VERSION }}
          registry-url: 'https://registry.npmjs.org'
          run-install: 'false'

      - name: Install dependencies
        if: inputs.release_type != 'deprecate'
        run: npm ci
```
