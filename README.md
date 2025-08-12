# Auto-Assign GitHub Issues to Copilot When Your CI Fails

![Copilot assigned actions](assets/copilot_auto_assign_issues.gif)

**TLDR:** Set up a GitHub Actions workflow that automatically creates issues when builds fail and assigns them to GitHub Copilot for AI-powered fixes. This way, you can focus on higher-level tasks while Copilot handles the debugging.

**Assumption:** It's Monday morning. Your GitHub Actions workflow failed overnight, and now you're faced with manually creating issues, parsing error logs, and tracking down the root cause. Sound familiar?

There's a better way. You can automate the entire process: when your GitHub Actions fail, automatically create an issue and let Copilot fix it for you. In the best case, it immediately suggests the correct fix. In many cases, it will give you a starting point for the fix which you can iterate on. No more manual issue creation, no more forgotten build failures.

## Why This Actually Matters
**Before:** CI fails → You manually create an issue → It sits in your backlog → Someone eventually reads the workflow logs → Replicates the issue locally → Debugs and fixes

**After:** CI fails → Issue created automatically with context → Copilot analyzes the failure → Suggests a fix or starting point → Quick review and merge

The difference? Your build failures get addressed automatically when they occur, saving you precious debugging time.

Here's how to set it up.

## What You'll Need

- A Github account. You can register one from [github.com](https://github.com).
- A GitHub repository, create one [here](https://github.com/new).
- Enable GitHub Copilot in your account. This allows Microsoft's AI to work on issues and create pull requests automatically.
- A GitHub Actions workflow. You can copy the code from [.github/workflows/copilot-create-issue.yml](.github/workflows/copilot-create-issue.yml)

## Setup Steps

Before we can assign issues to Copilot, you need to make sure GitHub Copilot is properly enabled and configured for your repository.

![Enable coding agent](assets/enable_coding_agent.gif)

## Step 1: Enable GitHub Copilot

![Enable Copilot](assets/step_1_enable_copilot.png)

First, ensure you have GitHub Copilot enabled for your account. If you don't have access yet, you'll need to subscribe to GitHub Copilot in your account settings.

## Step 2: Enable the Coding Agent

![Enable Coding Agent](assets/step_2_enable_coding_agent.png)

Next, enable the GitHub Copilot coding agent feature. This allows Copilot to work on issues and create pull requests automatically.

## Step 3: Allow the Coding Agent Access

![Allow Coding Agent](assets/step_3_allow_coding_agent.png)

Finally, make sure to allow the coding agent to work on your repositories. This permission is required for Copilot to create and update pull requests on your behalf. In this case, I chose to allow for select repositories to filter its access.

## Step 4: Create a Personal Access Token

<img src="assets/step_4_developer_setings.png" alt="Developer Settings" width="600"/>

Go to your GitHub **Developer Settings**. You need a fine-grained PAT because the workflow needs to create issues automatically.

![Create PAT](assets/step_5_create_pat.png)

Create a new **fine-grained Personal Access Token** with these permissions:
- **Read and Write access to Issues**
- **Read access to Repository**

*Note: You can't create fine-grained PATs via CLI. Has to be done in the UI.*

## Step 5: Configure Repository Secrets

<img src="assets/step_6_repo_secrets.png" alt="Developer Settings" width="600"/>

Navigate to your repository's Settings > Secrets and variables > Actions to configure the secrets needed for the workflow.

<img src="assets/step_7_new_secret.png" alt="Developer Settings" width="600"/>

Click "New repository secret" to add the personal access token you created.

<img src="assets/step_8_pat_secret.png" alt="Developer Settings" width="600"/>

Create a secret named `COPILOT_ASSIGN_PAT` and paste the Personal Access Token you created in Step 4. This secret is crucial because the workflow needs to authenticate with GitHub's GraphQL API to create issues and assign them directly to the Copilot agent.

**Why GraphQL instead of REST?** The GitHub REST API doesn't currently support assigning issues to the Copilot agent since it's not recognized as a traditional collaborator. The GraphQL API provides the necessary endpoints to find and assign issues to Copilot's bot ID programmatically.

## Step 6: Add the Workflow File

![Workflow File](assets/step_9_workflow_file.png)

Create the GitHub Actions workflow file at `.github/workflows/copilot-create-issue.yml`. The workflow uses the `COPILOT_ASSIGN_PAT` secret (which contains the PAT token we created in Step 4) to authenticate with GitHub's GraphQL API. 

**Technical Note:** The workflow specifically uses GraphQL because the GitHub REST API doesn't yet support creating issues assigned to the Copilot agent. The GraphQL implementation:
1. Queries for the Copilot bot ID using the repository's `suggestedActors` 
2. Creates the issue with the Copilot agent pre-assigned using the `createIssue` mutation
3. Uses your PAT token for authentication to perform these operations

You can copy the complete workflow code from [the repository](.github/workflows/copilot-create-issue.yml).

## Step 7: Test Your Setup

Before waiting for a real failure, test your workflow by intentionally triggering a failure:

1. Modify your workflow to include a step that will fail (like `run: exit 1`)
2. Push to your repository 
3. Watch the workflow fail and automatically create an issue
4. Verify the issue is assigned to `@github-copilot`
5. Remove the test failure and push again

## Step 8: How the Automation Works

Once your setup is complete, here's what happens when your build fails:

![Failed Workflow](assets/step_10_failed_workflow.png)

Your GitHub Actions workflow fails (happens to the best of us).

![On Failure](assets/step_11_on_failure.png)

The `on: failure` trigger activates your issue creation workflow.

![Issue Created](assets/step_12_issue_created.png)

A new GitHub issue is automatically created with:
- Workflow name and run number
- Failure timestamp  
- Link to the failed run
- Error context

![Open Issue](assets/step_13_open_issue.png)

The issue appears in your Issues tab, ready for action.

## Step 9: Copilot Takes Action

Since the issue was automatically assigned to Copilot in the previous step, the AI immediately begins analyzing the failure:

![PR Created](assets/step_14_pr_created.png)

Copilot analyzes the failure and creates a pull request to fix it.

![Copilot Started](assets/step_15_copilot_started.png)

The AI examines your codebase and the specific failure context.

![Copilot Session](assets/step_16_copilot_session.png)

You can chat with Copilot about the issue to understand the problem better.

![Copilot Action](assets/step_17_copilot_action.png)

Copilot proposes specific code changes to resolve the workflow failure.

## Step 10: Review and Merge

The rest follows standard GitHub flow:

![PR Updated](assets/step_18_pr_updated.png)

The pull request gets updated with Copilot's proposed fix.

![PR Changes](assets/step_19_pr_changes.png)

You review the changes to make sure they look reasonable.

![Ready for Review](assets/step_20_ready_for_review.png)

Mark the PR as ready for review.

![Approve](assets/step_21_approve.png)

Get approval from your team (or approve it yourself if you're feeling brave).

![Merged](assets/step_22_merged.png)

Merge the fix.

![Issue Closed](assets/step_23_issue_closed.png)

The original issue automatically closes. Done.

## Pro Tips

- **Don't skip the review step.** Copilot is smart, but it's not infallible.
- **Customize your issue templates** to include more context about failures.
- **Set up notifications** so you know when new auto-issues are created.
- **Use this for tests, not just builds.** Failed tests often have obvious fixes that Copilot can handle.

The best part? Once it's set up, you literally never think about it again. Your CI failures just... get fixed automatically.

*This works particularly well for common failure patterns: dependency updates, linting errors, simple test failures, and configuration issues. For complex bugs, you'll still need human intervention, but this handles the tedious stuff.*

## Who am I?

<img src="https://media.licdn.com/dms/image/v2/C4E03AQFnI0ilOq4ocQ/profile-displayphoto-shrink_800_800/profile-displayphoto-shrink_800_800/0/1636372591786?e=1758153600&v=beta&t=F6GTcHGTHqDNgPBngasObCUmen4YA-4zDr116r6ClXU" alt="Developer Settings" width="200"/>

I'm Lasse Benninga, a Data & Analytics Engineer with 7+ years of experience building data platforms and automation pipelines across AWS, GCP, and Azure. I've worked with companies like KLM Royal Dutch Airlines, Vattenfall, and ANWB, where I've architected and implemented scalable data solutions using everything from managing cloud infrastructure with Terraform to building data warehouses with Snowflake and dbt.

I believe building robust CI/CD pipelines and automation solutions is essential for streamlining development, reducing debugging time, and saving costs for clients. This GitHub Actions workflow was born out of the need to automate issue tracking for build failures, ensuring problems are detected and addressed quickly with AI-powered suggestions.

You can find more of my work on [GitHub](https://github.com/lassebenni) or connect with me on [LinkedIn](https://www.linkedin.com/in/lasse-benninga-a462b194/).