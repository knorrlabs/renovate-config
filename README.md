# renovate-config

Shared [Renovate](https://docs.renovatebot.com) configuration presets for `knorrlabs` and `etknorr` repositories.

## Usage

Reference the preset from a repository's `renovate.json`:

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["github>knorrlabs/renovate-config"]
}
```

Renovate resolves a bare `github>owner/repo` reference to `default.json` on the default branch, so no filename is needed.

Repositories under `knorrlabs` also pick this up automatically at onboarding: Renovate checks the parent org for a repository named `renovate-config` containing `default.json`. Repositories under `etknorr` sit outside that org, so they must extend the preset explicitly.

## What the default preset does

- Holds a release for seven days before it is eligible, so a compromised publish is usually yanked first. Security fixes skip that quarantine.
- Groups GitHub Actions and container images so they land together instead of one PR each.
- Requires dashboard approval before a major update is opened.
- Automerges non-major updates (after the seven-day quarantine) for two low-risk cases: trusted `actions/*` GitHub Actions, and dependencies under a repo's `docs/**` path. Everything else lands through a reviewed pull request.

## Optional add-on presets

Extra policy that not every project wants, kept out of `default.json` so a repo opts in deliberately. Reference alongside the default preset:

```json
{
  "extends": [
    "github>knorrlabs/renovate-config",
    "github>knorrlabs/renovate-config:ignition-automerge"
  ]
}
```

- **`ignition-automerge`** — automerges patch-level updates (after the standard quarantine) to any dependency with "ignition" in its name: the platform Docker image, `ignition-api-stubs`, `bwdesigngroup/ignition-docker`, and similar. Minor updates still need review, since Ignition's 8.x line moves mostly at the patch level. Only extend this in a repo that actually tracks an Ignition dependency.

## Local overrides

A repository keeps its own `renovate.json` for anything specific to it, such as language grouping or a version pin, alongside the `extends`. See `stoker-operator` for an example.
