# approval-gate-playground

Public, disposable, and here for one reason: **GitHub Free can only configure
environments on public repositories**, so this is the only place the approval
gate in `SkyblockRealm/workspace/.github/workflows/example-approval-gate.yml`
can actually be exercised.

The workflow is a fake deploy. It touches no server and holds no secrets.

## What it proves

1. The `plan` job runs immediately, with no gate.
2. The `deploy` job stops at *Waiting for approval* because it names
   `environment: production`, which has a required reviewer.
3. After approval it runs, reads back who approved from the Actions approvals
   API, and writes the audit row into the run summary.

## The settings that make it work

The `production` environment needs **Required reviewers** configured. Naming an
environment with no rules is a no-op — the job runs immediately and reports
success, which looks identical to a working gate from the outside. Verify:

```sh
gh api repos/SkyblockRealm/approval-gate-playground/environments/production \
  --jq '{name, rules: .protection_rules}'
```

`prevent_self_review` is **off here on purpose**, so the one account in the test
can dispatch and approve its own run. In a real repository it should be on.

## Running it

```sh
gh workflow run example-approval-gate.yml \
  -f target=islands-01 -f artefact=skyblock-3f2a9c1b7d0e -f dry_run=false

gh run list --workflow example-approval-gate.yml --limit 1
```

While it waits, `gh api repos/.../actions/runs/<id>/pending_deployments` lists
the gate; approve it in the UI or with a `POST` to the same path.
