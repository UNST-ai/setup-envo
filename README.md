# setup-envo

Give your CI jobs — and the coding agents running in them — their working
environment in one step: secrets, agent skills, and model-provider auth,
pulled from [envo.sh](https://envo.sh) as an encrypted pack and decrypted
only inside the job.

## The problem

Cloud coding agents and CI jobs wake up in empty sandboxes. No `GITHUB_TOKEN`
for private pushes, no `DATABASE_URL` for integration tests, no npm/pip
registry credentials, no provider auth for sub-agents. So they write code
they can't run, test, or ship.

## Usage

```yaml
- uses: UNST-ai/setup-envo@v1
  env:
    ENVO_TOKEN: ${{ secrets.ENVO_TOKEN }}   # scoped, pull-only agent token
    ENVO_KEY: ${{ secrets.ENVO_KEY }}       # your pack's encryption key
  with:
    ref: myproject/ci
# Later steps now have every env var from the pack (masked in logs),
# skills on disk, and provider credentials in place.
- run: bun test          # DATABASE_URL, NPM_TOKEN, etc. are present
```

## Inputs

| input | default | |
|---|---|---|
| `ref` | — | environment to materialize: `<project>/<environment>` |
| `dir` | `.` | directory to materialize into |
| `harness` | pack default | skill placement: `claude-code`, `codex`, `opencode`, `cursor`, … |
| `export-env` | `true` | export env vars to `GITHUB_ENV` for later steps (values masked) |

## Security model

- The registry stores ciphertext only (AES-256-GCM, context-bound). Decryption
  happens inside your job with `ENVO_KEY`, which Envo never sees.
- Use a **pull-only token scoped to one environment** for CI, and a
  **dedicated key** for CI environments — never your personal master key.
- Exported values are masked in workflow logs via `::add-mask::`.

Spec and CLI: https://github.com/UNST-ai/envo · https://envo.sh
