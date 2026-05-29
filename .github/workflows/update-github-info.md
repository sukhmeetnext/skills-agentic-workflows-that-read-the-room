---
name: Update GitHub Info
on:
  schedule:
    - cron: '0 6 * * *' # daily at 06:00 UTC
  workflow_dispatch: {}

# Agentic workflow frontmatter
# This workflow is an agentic (gh-aw) workflow that reads local notes,
# fetches GitHub blog pages, updates site/content/github-info.md, and
# opens a pull request for Mona to review. It uses safe outputs and create-pull-request
---

## Metadata

- title: Update GitHub Info from blog and notes
- description: |
    Read local notes and the GitHub Blog to refresh site/content/github-info.md,
    then propose changes via a pull request for Mona to review.

## Inputs

- name: run-mode
  description: 'Run mode: "daily" or "on-demand" (workflow_dispatch)'
  required: false
  default: 'daily'

## Tools

# Grant the agent edit access to the repo via the `repo` tool and allow
# web fetch + read-file + create-pull-request. Use safe-outputs so the agent
# returns the proposed file content rather than writing directly to `main`.

- name: repo
  permissions:
    contents: write

- name: web-fetch
  permissions:
    network: [https://github.blog/*]

- name: read-file
  permissions:
    path: [notes/mona-notes.md]

- name: create-pull-request
  permissions:
    safe-outputs: true

## Steps

The agent should perform the following steps in order:

1. Read the local file [notes/mona-notes.md](notes/mona-notes.md).
2. Web fetch https://github.blog/latest/ and https://github.blog/changelog/.
3. Combine insights from the notes and fetched pages to produce an updated
   version of `site/content/github-info.md` (preserve existing structure and
   frontmatter; update the content body and any 'Latest' or 'Changelog' sections).
4. Produce the updated file content as a safe output named `updated_file`.
5. Use the `create-pull-request` tool to open a PR against `main` with the
   branch name `update/github-info-<timestamp>`, target branch `main`, title
   `chore: update GitHub info (automated)`, and body describing the changes.

## Usage / Run Notes

This workflow runs daily (schedule) and can be triggered manually via
workflow_dispatch. The agent has edit access through the `repo` tool
configuration above, but changes are proposed using `create-pull-request`
with `safe-outputs` so updates require Mona to merge.

## Agent Prompt

You are a safe, sandboxed agent. Follow these constraints strictly:

- Do not write directly to the `main` branch.
- Use the `read-file` tool to read `notes/mona-notes.md`.
- Use the `web-fetch` tool to GET the two URLs exactly:
  - https://github.blog/latest/
  - https://github.blog/changelog/
- Produce the new content for `site/content/github-info.md` and return it as
  a safe output named `updated_file` containing the full file contents.
- Use `create-pull-request` to open a PR with the updated file applied on a
  new branch. Include a concise summary of what changed and sources used.

## Outputs

- updated_file: |
    The full contents of the updated `site/content/github-info.md` file.
