# GitHub Actions Pipeline

This directory contains GitHub Actions workflows for automated scraping and data collection from Boshamlan.

## Workflows

### 1. `pipeline.yml` - Main Pipeline Orchestrator
**Purpose:** Coordinates the execution of all scraping workflows and tracks their status.

**Schedule:** Daily at 12:00 AM UTC

**Workflow:**
1. Runs unified scraper workflow (`scraper.yml`)
   - Properties Scraper runs first
   - Offices Scraper runs after properties complete
2. Updates `status.json` with execution results

**Outputs:**
- Creates/updates `status.json` in the repository root with:
  - Execution date and timestamp
  - Overall pipeline status (success/failed)
  - Individual scraper statuses
  - Pipeline run ID and number

**Manual Trigger:** Available via GitHub Actions tab

---

### 2. `scraper.yml` - Unified Scraper Workflow
**Purpose:** Single workflow that runs both properties and offices scrapers sequentially.

**Can be triggered by:**
- Pipeline workflow
- Schedule (daily at 12:00 AM UTC)
- Manual trigger

**Jobs:**
1. **scrape-properties** - Scrapes property listings
   - Scrapes all property categories and subcategories
   - Filters data from yesterday onwards
   - Uploads images to S3
   - Generates Excel files with data
   - Uploads Excel files to S3

2. **scrape-offices** - Scrapes office information (runs after properties)
   - Scrapes office information from agents page
   - Collects listings from each office
   - Retrieves view counts for each listing
   - Uploads images to S3
   - Generates Excel files per office
   - Uploads Excel files to S3

**Performance Optimization:**
- **Pip caching**: Python packages are cached automatically
- **Playwright browser caching**: Chromium browser (~130MB) is cached between runs
- Cache invalidation: Only re-downloads when `requirements.txt` changes
- Typical speedup: ~2-3 minutes saved per workflow run

---

## Status Tracking

The pipeline automatically maintains a `status.json` file in the repository root.

### Status.json Format
```json
{
  "date": "2026-03-15 00:30:45 UTC",
  "timestamp": "2026-03-15T00:30:45Z",
  "overall_status": "success",
  "workflows": {
    "properties": {
      "status": "success",
      "workflow": "scraper.yml"
    },
    "offices": {
      "status": "success",
      "workflow": "scraper.yml"
    }
  },
  "pipeline_run_id": "1234567890",
  "pipeline_run_number": "42"
}
```

### Status Values
- `success` - Workflow completed successfully
- `failed` - Workflow failed or was cancelled
- `skipped` - Workflow was skipped

---

## Configuration

### Required Secrets
Configure these secrets in your GitHub repository settings:

- `AWS_ACCESS_KEY_ID` - AWS access key for S3 uploads
- `AWS_SECRET_ACCESS_KEY` - AWS secret key for S3 uploads

### Caching
The workflows use GitHub Actions cache to speed up execution:

**What's cached:**
- Python packages (via `cache: 'pip'` in setup-python)
- Playwright Chromium browser (~130MB)

**Cache benefits:**
- Pip packages: ~30 seconds saved
- Playwright browser: ~1-2 minutes saved per job
- Total savings: ~2-4 minutes per pipeline run

**Cache invalidation:**
- Pip cache: Updates when `requirements.txt` changes
- Playwright cache: Updates when `requirements.txt` changes
- GitHub cache retention: 7 days (refreshed on each use)

---

## Manual Execution

You can manually trigger any workflow:

1. Go to **Actions** tab in GitHub
2. Select the workflow you want to run:
   - **Daily Scraper Pipeline** - runs the entire pipeline
   - **Daily Scraper** - runs both scrapers (properties then offices)
3. Click **Run workflow**
4. Select branch (usually `main`)
5. Click **Run workflow** button

---

## Monitoring

### Check Pipeline Status
- View latest `status.json` in repository root
- Check GitHub Actions tab for workflow runs
- Download status artifacts (retained for 30 days)

### Logs
All workflows provide detailed logs:
- Setup and dependency installation
- Scraping progress and results
- S3 upload status
- Error messages and stack traces

---

## Workflow Dependencies

```
pipeline.yml
└── scraper.yml
    ├── scrape-properties (job 1)
    └── scrape-offices (job 2, runs after job 1)
        └── status.json update (runs after both, always)
```

---

## Troubleshooting

### Pipeline Failed
Check `status.json` to identify which workflow failed, then:
1. Review workflow logs in GitHub Actions
2. Check AWS credentials and permissions
3. Verify website accessibility
4. Review error messages in logs

### Status.json Not Updated
- Check if update-status job ran (should run even if workflows fail)
- Verify GitHub Actions has write permissions
- Check git push logs for errors

### Individual Workflow Failed
Each workflow can be run independently for testing:
1. Go to Actions tab
2. Select the failed workflow
3. Click "Run workflow" to retry
4. Check logs for specific error details
