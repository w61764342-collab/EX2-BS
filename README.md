# EX2-BS - Boshamlan Data Scraper

Automated web scraping system for collecting real estate data from Boshamlan.com with AWS S3 storage and GitHub Actions pipeline.

## 🚀 Features

- **Automated Daily Scraping** - Properties and offices data collected daily
- **GitHub Actions Pipeline** - Orchestrated workflow execution
- **S3 Storage** - Date-partitioned data storage in AWS S3
- **Status Tracking** - Automatic status.json updates in repository
- **Human-Like Behavior** - Random delays and user-agents to avoid detection
- **Comprehensive Logging** - Detailed execution logs and status reporting

## 📋 Project Structure

```
EX2-BS/
├── .github/workflows/       # GitHub Actions workflows
│   ├── pipeline.yml        # Main pipeline orchestrator
│   ├── scraper.yml         # Unified scraper (properties + offices)
│   ├── README.md          # Workflows documentation
│   └── PIPELINE_FLOW.md   # Pipeline flow diagram
├── properties/             # Properties scraping module
│   ├── CategoryScraper.py
│   ├── PropertyCardScraper.py
│   ├── S3Uploader.py
│   └── main_s3.py
├── offices/               # Offices scraping module
│   ├── OfficeScraper.py
│   ├── OfficeS3Uploader.py
│   └── main_offices_s3.py
├── scraper_utils.py       # Shared utilities (delays, user-agents)
├── status.json           # Pipeline execution status
└── requirements.txt      # Python dependencies
```

## 🔄 Pipeline Workflow

The GitHub Actions pipeline runs daily at 12:00 AM UTC:

1. **Unified Scraper** (`scraper.yml`) runs both:
   - Properties Scraper (job 1)
   - Offices Scraper (job 2 - runs after properties)
2. **Status Update** - Updates status.json with execution results

### Pipeline Status

Check the latest pipeline status in [`status.json`](status.json):

```json
{
  "date": "2026-03-15 00:30:45 UTC",
  "overall_status": "success",
  "workflows": {
    "properties": {"status": "success"},
    "offices": {"status": "success"}
  }
}
```

## 🛠️ Setup

### Prerequisites
- Python 3.11+
- AWS account with S3 access
- GitHub repository with Actions enabled

### Local Development

1. **Clone the repository**
   ```bash
   git clone <repository-url>
   cd EX2-BS
   ```

2. **Install dependencies**
   ```bash
   pip install -r requirements.txt
   playwright install chromium
   ```

3. **Configure AWS credentials**
   ```bash
   export AWS_ACCESS_KEY_ID="your-key"
   export AWS_SECRET_ACCESS_KEY="your-secret"
   export AWS_DEFAULT_REGION="us-east-1"
   ```

4. **Run scrapers**
   ```bash
   # Properties scraper
   cd properties && python main_s3.py
   
   # Offices scraper
   cd offices && python main_offices_s3.py
   ```

### GitHub Actions Setup

1. **Add GitHub Secrets**
   - Go to Settings → Secrets and variables → Actions
   - Add the following secrets:
     - `AWS_ACCESS_KEY_ID`
     - `AWS_SECRET_ACCESS_KEY`

2. **Enable GitHub Actions**
   - Actions should be enabled by default
   - Pipeline runs automatically daily at 12:00 AM UTC

3. **Manual Trigger**
   - Go to Actions tab
   - Select "Daily Scraper Pipeline" (full pipeline)
   - Or select "Daily Scraper" (just scrapers, no status update)
   - Click "Run workflow"

## 📊 Data Output

### S3 Structure

Data is stored in AWS S3 with date partitioning:

```
s3://data-collection-dl/
└── boshamlan-data/
    ├── properties/
    │   └── year=2026/month=03/day=15/
    │       ├── excel files/
    │       │   ├── rent.xlsx
    │       │   ├── sale.xlsx
    │       │   └── exchange.xlsx
    │       └── images/
    │           ├── rent/
    │           ├── sale/
    │           └── exchange/
    └── offices/
        └── year=2026/month=03/day=15/
            ├── excel files/
            │   └── offices_*.xlsx
            └── images/
                └── [office_name]/
```

### Excel Files

**Properties**: Each category (rent/sale/exchange) has sheets for subcategories:
- عقارات, شقة, بيت, أرض, عمارة, شاليه, مزرعة, تجاري

**Offices**: Single file with all offices and their listings

## 🤖 Human-Like Scraping

To avoid detection, the scrapers implement:

- **15+ Random User-Agents** - Chrome, Firefox, Safari, Edge
- **Variable Delays** - Random timing between requests
  - Page loads: 1.5-3.5 seconds
  - Between items: 1.0-3.0 seconds  
  - Image downloads: 0.3-1.0 seconds
- **Random Headers** - Accept-Language, DNT, viewport sizes
- **Browser Fingerprinting** - Randomized locale, timezone, viewport

See [`scraper_utils.py`](scraper_utils.py) for implementation details.

## 📖 Documentation

- [Workflows Documentation](.github/workflows/README.md)
- [Pipeline Flow Diagram](.github/workflows/PIPELINE_FLOW.md)
- [Properties README](properties/README.md)
- [Offices README](offices/README.md)

## 🔍 Monitoring

### Check Pipeline Status
- View [`status.json`](status.json) in repository
- Check GitHub Actions tab for workflow runs
- Download status artifacts (30-day retention)

### Logs
All workflows provide detailed logs in GitHub Actions

## 🐛 Troubleshooting

See [Workflows Documentation](.github/workflows/README.md#troubleshooting) for common issues and solutions.

## 📝 License

[Add your license information here]

## 👥 Contributors

[Add contributors information here]
