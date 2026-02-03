# agent_evo

## Git hooks (secret scanning)

This repo includes a versioned pre-commit hook at `.githooks/pre-commit`.

Enable it for this clone:

```sh
git config core.hooksPath .githooks
```

Install `gitleaks` (recommended) for full scanning:

```sh
brew install gitleaks
```

If `gitleaks` isn’t installed, the hook falls back to a small set of high-signal secret patterns.
