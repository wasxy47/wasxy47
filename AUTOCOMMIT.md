# Daily Auto-Commit — Complete Setup, Run, and Troubleshooting Guide

This document explains exactly how the automated daily commit system is set up, how it runs, how to make sure your contributions appear on your GitHub graph, and how to troubleshoot issues.

---

## What this does

- Makes 1–10 small commits each day
- Writes an activity log file under `logs/` (one per day)
- Runs automatically at 00:00 UTC (5:00 AM Karachi)
- Can be triggered manually from the Actions tab
- Attributes commits to you so they count on your contribution graph

---

## Schedule and Timezone

- Cron schedule: `0 0 * * *` (00:00 UTC, daily)
- Karachi (PKT, UTC+5) equivalent: 5:00 AM local time

GitHub Actions may start a few minutes after the scheduled time, but by 5:10 AM PKT it should have run.

---

## Current configuration (you already set this up)

- Default branch: the repository’s default (usually `main`)
- Actions enabled for the repository
- Author attribution variables created:
  - `AUTHOR_NAME`: `wasxy47`
  - `AUTHOR_EMAIL`: `167606264+wasxy47@users.noreply.github.com`
- Workflow file updated to:
  - Check out the default branch explicitly
  - Push safely even from a detached HEAD
  - Add “Co-authored-by” with your name and email so contributions count

---

## Files created daily

- A log file for the current date:
  - `logs/activity-YYYY-MM-DD.log`
- Example entries:
  ```
  [2026-01-25 05:00:03] Activity #1 - Automated commit to maintain streak
  [2026-01-25 05:00:06] Activity #2 - Automated commit to maintain streak
  ```
- Multiple commits may be created (1–10) with short delays between them

---

## How to set up (step-by-step)

1) Confirm default branch
- Repository → Settings → Branches → Default branch
- Note the name (usually `main`)

2) Ensure Actions are enabled
- Settings → Actions → General
- Permissions should allow repository workflows
- Workflow permissions can be “Read and write” (or keep `contents: write` inside the job)

3) Create author attribution variables
- Repository → Settings → Secrets and variables → Actions → Variables → New variable
- Add:
  - Name: `AUTHOR_NAME`, Value: `wasxy47` (or your full name)
  - Name: `AUTHOR_EMAIL`, Value: `167606264+wasxy47@users.noreply.github.com`

4) Update the workflow file
- Edit `.github/workflows/daily-commit.yml` with the content below

```yaml
name: Daily Auto Commit

on:
  schedule:
    # Runs daily at 12:00 AM UTC (5:00 AM Karachi)
    - cron: "0 0 * * *"
  workflow_dispatch: # Allows manual trigger

jobs:
  auto-commit:
    runs-on: ubuntu-latest
    permissions:
      contents: write

    env:
      BRANCH: ${{ github.event.repository.default_branch }}
      AUTHOR_NAME: ${{ vars.AUTHOR_NAME }}
      AUTHOR_EMAIL: ${{ vars.AUTHOR_EMAIL }}

    steps:
      - name: Checkout repository (default branch)
        uses: actions/checkout@v4
        with:
          fetch-depth: 0
          token: ${{ secrets.GITHUB_TOKEN }}
          ref: ${{ github.event.repository.default_branch }}
      
      - name: Configure Git (bot as committer)
        run: |
          git config --global user.name "github-actions[bot]"
          git config --global user.email "github-actions[bot]@users.noreply.github.com"
      
      - name: Make random commits (co-author you)
        run: |
          NUM_COMMITS=$((RANDOM % 10 + 1))
          echo "Making $NUM_COMMITS commits today"

          DATE=$(date +"%Y-%m-%d")
          
          for i in $(seq 1 $NUM_COMMITS); do
            TIMESTAMP=$(date +"%Y-%m-%d %H:%M:%S")
            LOG_FILE="logs/activity-${DATE}.log"
            echo "[$TIMESTAMP] Activity #$i - Automated commit to maintain streak" >> "$LOG_FILE"
            
            git add "$LOG_FILE"
            git commit -m "Daily activity update #$i on $DATE" -m "Co-authored-by: ${AUTHOR_NAME} <${AUTHOR_EMAIL}>"
            
            if [ $i -lt $NUM_COMMITS ]; then
              sleep $((RANDOM % 5 + 1))
            fi
          done
      
      - name: Push changes to default branch
        run: |
          # Works even if HEAD is detached
          git push origin HEAD:"$BRANCH"
```

5) Save changes
- Commit directly to the default branch (e.g., `main`)

---

## How to run manually (test immediately)

1) Go to the repository’s “Actions” tab
2) Open “Daily Auto Commit”
3) Click “Run workflow”
4) Select default branch (e.g., `main`)
5) Click “Run workflow”

After it finishes:
- Verify a new log file exists: `logs/activity-YYYY-MM-DD.log`
- Open the latest commit — you should see a line:
  ```
  Co-authored-by: wasxy47 <167606264+wasxy47@users.noreply.github.com>
  ```

This co-author attribution makes the commit count toward your personal contribution graph.

---

## Why your contribution graph might not update (and fixes)

- If commits are authored solely by `github-actions[bot]`, they don’t count for you.
- Adding `Co-authored-by: Your Name <Your Email>` makes it count.
- Your email must be your GitHub-linked email (noreply works if privacy is on).
- Graph updates can take a few minutes; refresh your profile.

---

## Troubleshooting

- Push fails (“detached HEAD”)
  - The workflow now uses `git push origin HEAD:"$BRANCH"`, which works even in detached HEAD.
- Permissions error
  - Ensure the job has `permissions.contents: write`, and Actions → Workflow permissions allow writes.
- Wrong branch
  - The workflow checks out `${{ github.event.repository.default_branch }}`. Ensure your workflow file is on the default branch.
- No changes detected
  - The workflow writes a log entry every run, guaranteeing a change to commit.
- Time didn’t match local
  - Cron runs at 00:00 UTC, which is 5:00 AM Karachi. Manual runs can be done any time from the Actions tab.

---

## Optional: Set yourself as the main author (instead of co-author)

If you prefer to be the author (not just co-author), use either:

- Configure Git with your variables:
  ```
  git config --global user.name "${AUTHOR_NAME}"
  git config --global user.email "${AUTHOR_EMAIL}"
  ```
  Then commit without the co-author line.

- Or set author per commit:
  ```
  git commit --author="${AUTHOR_NAME} <${AUTHOR_EMAIL}>" -m "Daily activity update..."
  ```

Co-authoring is recommended because it keeps the bot as committer while crediting you.

---

## Maintenance

- Logs can grow over time; you can periodically clean older files in `logs/`.
- To change the schedule, edit the cron (`"0 0 * * *"`). For example:
  - 1:30 AM UTC: `"30 1 * * *"`
- To reduce commits, change `NUM_COMMITS` logic, e.g. max 3:
  ```
  NUM_COMMITS=$((RANDOM % 3 + 1))
  ```

---

## Quick checklist

- [x] Actions enabled
- [x] Default branch confirmed
- [x] `AUTHOR_NAME` and `AUTHOR_EMAIL` variables created
- [x] Workflow updated to push from detached HEAD and add co-author
- [x] Manual run verified (log file + commit with co-author)
- [x] Wait a few minutes and refresh contribution graph

---

## Summary

You have fully set up the daily auto-commit system:
- It runs at 00:00 UTC (5:00 AM Karachi)
- It logs activity under `logs/`
- It adds you as co-author using:
  - `AUTHOR_NAME`: `wasxy47`
  - `AUTHOR_EMAIL`: `167606264+wasxy47@users.noreply.github.com`
- Your contributions should now appear on your GitHub graph after each run.
