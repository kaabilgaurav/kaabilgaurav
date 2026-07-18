# Workflow Publish Fix Report

## What changed

The manual `Commit & Push Changes` shell step in `.github/workflows/update-profile-art.yml` was replaced with:

```yaml
uses: stefanzweifel/git-auto-commit-action@v5
```

The action is limited to these generated files:

- `contrib-heatmap.svg`
- `gaurav-ascii.svg`
- `info-card.svg`
- `data/contributions.json`

It also uses GitHub Actions’ standard bot commit identity.

All manual Git publication logic was removed, including `git pull --rebase`, the retry loop, `sleep`, and `git rebase --abort` handling.

## Why the conflict happened

`data/contributions.json` is regenerated on every workflow run. The former workflow created a local commit and then ran `git pull --rebase origin main`. If `main` already contained another workflow-generated version of that same file, Git attempted to replay the local generated change on top of it and produced a content conflict.

## Why `git-auto-commit-action` is safer here

The action encapsulates the standard detect-change, commit, and push sequence instead of replaying a locally generated commit through custom rebase/retry shell logic. Restricting `file_pattern` ensures that only the four intended generated artifacts can be committed.

The existing workflow concurrency group serializes same-ref runs, and `paths-ignore` excludes those generated files from push-triggered runs. Together, they prevent overlapping publisher runs and generated-asset workflow loops.

## Preserved workflow behavior

- Daily scheduled execution remains enabled.
- Manual `workflow_dispatch` remains enabled.
- Push execution on `main` remains enabled.
- `paths-ignore`, `concurrency`, and `contents: write` permission remain unchanged.
- The asset generation steps and all source paths remain unchanged.

## Validation performed

- Parsed workflow YAML successfully.
- Confirmed the workflow uses `stefanzweifel/git-auto-commit-action@v5`.
- Confirmed no `git pull`, `git rebase`, retry loop, `sleep`, or manual Git commit/push commands remain in the workflow.
- Confirmed all four generated files exist and are tracked.
- Confirmed the four action `file_pattern` paths exactly match the intended generated files.
- Confirmed generated-only paths are still ignored by the push trigger, preventing infinite loops.
- Compiled all Python scripts successfully.
- Confirmed `README.md` has balanced fenced code blocks and valid local SVG image paths.
- Parsed `data/contributions.json` and all three SVG files successfully.
- Ran `git diff --check` successfully.

## Remaining risks

- The action still needs the repository’s `contents: write` permission, which the workflow already grants.
- An unrelated external process that changes the same generated files outside this workflow may still cause a push rejection; concurrency prevents conflicts between this workflow’s own runs but cannot control external writers.
- Contribution retrieval continues to rely on GitHub’s public HTML calendar and can be affected by markup changes or rate limits.

No commit or push was performed during this fix.
