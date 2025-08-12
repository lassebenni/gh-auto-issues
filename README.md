# gh-auto-issues

Blog to automatically create copilot issues from workflow failure.

## How it works

The repository contains a GitHub Actions workflow (`random-failure.yml`) that demonstrates automatic issue creation when a workflow job fails:

1. **failure-test job**: Intentionally fails with `exit 1` but has `continue-on-error: true` to prevent the overall workflow from failing
2. **on-failure job**: Runs when the failure-test job fails (`if: always() && needs.failure-test.result == 'failure'`) and creates a GitHub issue assigned to Copilot

This setup allows the workflow to:
- Detect when jobs fail
- Automatically create issues for investigation
- Assign those issues to Copilot for automated resolution
- Show overall workflow status as successful (preventing noise in the workflow status)
