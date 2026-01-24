# Daily Auto-Commit Feature

This repository includes an automated daily commit system to maintain GitHub contribution streaks.

## How It Works

The system uses GitHub Actions to automatically create commits every day at midnight UTC (00:00).

### Features

- **Random Commits**: Makes 1-10 random commits each day
- **Activity Logs**: Creates timestamped log entries in the `logs/` directory
- **Automated**: Runs automatically via GitHub Actions scheduled workflow
- **Manual Trigger**: Can be manually triggered via GitHub Actions UI

### Workflow Details

- **Schedule**: Runs daily at 12:00 AM UTC
- **Trigger**: Automatic via cron schedule or manual via `workflow_dispatch`
- **Commits**: Each day generates a random number of commits (1-10)
- **Log Files**: All activity is logged to `logs/activity-YYYY-MM-DD.log`

### Files Created

```
logs/
└── activity-2026-01-24.log
```

Each log file contains entries like:
```
[2026-01-24 13:13:46] Activity #1 - Automated commit to maintain streak
[2026-01-24 13:13:52] Activity #2 - Automated commit to maintain streak
...
```

### Manual Triggering

You can manually trigger the workflow:

1. Go to the repository's **Actions** tab
2. Select **Daily Auto Commit** workflow
3. Click **Run workflow**
4. Choose the branch (usually `main`)
5. Click **Run workflow** button

### Workflow Configuration

The workflow is defined in `.github/workflows/daily-commit.yml` and includes:

- Automatic checkout of repository
- Git configuration for bot commits
- Random commit generation (1-10 per day)
- Automatic push to the default branch

### Benefits

- ✅ Maintains GitHub contribution streak automatically
- ✅ Shows consistent activity in contribution graph
- ✅ Fully automated - no manual intervention needed
- ✅ Transparent - all commits are logged and traceable
- ✅ Flexible - can be manually triggered anytime

## Notes

- The workflow uses `github-actions[bot]` as the committer
- Commits are made to the default branch (main/master)
- The `logs/` directory will accumulate daily log files
- Old logs can be cleaned up manually if desired
